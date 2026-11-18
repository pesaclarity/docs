# 0021. Small language models on rayo, on the CPU

Date: 2026-11-18 · Status: Proposed · Builds on [0020](0020-rayo-our-own-inference-engine.md)

## Context

rayo ([0020](0020-rayo-our-own-inference-engine.md)) runs the Ask models: a 240k-parameter
CNN and a 532k-parameter transformer. The next step is small language models (SLMs):
decoder-only transformers of about 100M to 1.5B parameters, run on the phone, offline.

- **Where they run.** Android phones first, including phones without Google Mobile Services
  (Huawei and HarmonyOS, de-Googled ROMs, F-Droid builds), many with 2 to 3 GB of RAM and
  32-bit armv7 CPUs. Then the desktops rayo already builds for (Linux, macOS on Intel and
  Apple silicon), for trying models. CPU only: no GPU backend, no NNAPI, nothing from Google.
- **What it costs.** A 135M model is 540 MB in float32 and the phone has other things to do.
  Memory and battery decide more than speed.
- **What stays true.** rayo is general and portable for other users: no per-model code
  paths and no model constants in the library. C89, built from source by the Dart hook, as in
  0020. A run allocates nothing.
- **Candidate models and their licences.**
  - SmolLM2 (135M, 360M, 1.7B): Apache-2.0.
  - Qwen2.5 (0.5B, 1.5B): Apache-2.0. Qwen2.5-3B and larger use Qwen's own research licence:
    not candidates.
  - Phi-3.5-mini and Phi-4-mini (Microsoft): MIT, but 3.8B, too large for a phone of 2 to
    3 GB.
  - Llama 3.2 1B (Meta): the Llama Community Licence: "Built with Llama" attribution, an
    acceptable-use policy, a 700M-monthly-user limit, and the name rules. Not open source;
    usable only after a separate decision.
  - Google's models (Gemma and others): not used, by rule ([0019](0019-on-device-ml-without-google.md)).

## Options

- **(a) llama.cpp (ggml)**: MIT, the reference for running GGUF models on CPUs. Its CPU backend
  and GGUF reader are C++ (0020's reason for leaving ggml), it is about 4 MB of vendored code
  that changes weekly, and it would be a second engine next to rayo.
- **(b) MLC LLM (TVM)**: Apache-2.0. Models compiled ahead of time per device; a large C++
  runtime and a Python compiler stack; aimed at GPUs.
- **(c) ExecuTorch (PyTorch)**: BSD. C++, models exported with torch; XNNPACK on the CPU.
  Heavy to build from source for F-Droid.
- **(d) MediaPipe LLM Inference, Gemini Nano, NNAPI**: Google's: ruled out.
- **(e) rayo's own path**: the few ops these models add, 4-bit weights with kernels per
  architecture, mapped weights, threads, a tokeniser and a sampler, all in C89 as the rest
  of rayo. One engine, one build, the same tests and the same size budget.

## Decision

**Option (e): rayo runs decoder-only language models itself, described by each model's own
configuration, on the CPUs it already supports.** The spike is
[pesaclarity/rayo#6](https://github.com/pesaclarity/rayo/pull/6).

- **One generic decoder, driven by the model's config.json.** `rayo-convert` reads a
  Hugging Face model (config.json and safetensors) and writes a rayo graph of general ops.
  Everything model-specific is a number in the file:
  - the layers, heads, key/value heads and head size;
  - RoPE's theta, scaling (none, linear, Llama 3's, dynamic up to the original length) and
    rotated dimensions; the scaling is a frequency table the converter writes, so rayo has no
    code per scheme;
  - the norm (RMSNorm) and its epsilon; the gate's activation (SiLU, GELU);
  - a tied embedding (one weight named by two nodes), optional q, k and v biases (Qwen2), and
    fused qkv or gate-up projections (split by the converter);
  - the vocabulary, the stop tokens, the BOS rule and the chat template.
  The tensors are found by the names the Llama family shares (Llama, Mistral, Qwen2, SmolLM).
  What does not fit is refused, naming it. A model that needs another op gets a general one.
- **New ops**, each with a generic kernel, a NumPy reference and tests: `rms_norm`, `rope`
  (half or interleaved), `silu`, `swiglu`, grouped-query attention in `attention`
  (`kv_heads` below `heads`), `gather` from 4-bit and per-row int8 tables (the tied embedding).
- **Weight-only 4-bit groups ("int4"), and int8 kept.**
  - Layout: groups of 32 along each row, a float32 scale each. Byte j of a group holds value j
    in its low four bits and value j + 16 in its high four, each stored plus 8 (0 to 15 for
    -8 to 7). Real value = scale × (stored − 8). The converter maps each group's value of
    largest magnitude to −8. 5 bits a weight.
  - Activations are quantised to int8 per group on the fly; products are exact in int32 per
    four values, then scaled in float into eight lanes, added in a fixed order. Every kernel
    gives the same floats, bit for bit: generic C, SSE4.1, AVX2, NEON, SDOT, chosen at run
    time as 0020's are.
  - int8 weights (one scale per row) stay, run by 0020's kernels.
  - A session may keep the key/value cache in int8, a float32 scale per token and head:
    a quarter of float32's memory ([rayo#7](https://github.com/pesaclarity/rayo/pull/7)).
  - Optionally, the group scales in float16 (4.5 bits a weight, the converter's default
    since rayo#7) and 3-bit groups for matrices chosen by their role in the graph
    (`--q3 mlp`: 3.5 bits there).
- **Weights read in place.** `rayo_model_load` maps the file behind a small platform layer
  (POSIX mmap, Win32 MapViewOfFile, fread where neither exists, so a plain C89 build works).
  Pages are read as they are touched and can be dropped under memory pressure.
  - The file stays PCWT version 1: `int4` is a new dtype, which an old reader refuses by name
    rather than misreads. The converter starts the data and every tensor at 64-byte offsets
    (the header is padded), for aligned mapped reads; readers need only 4-byte alignment.
  - Offsets are 32-bit (C89), so a file holds at most 2 GiB: a 0.5B model fits in float32, a
    1.5B model only in int8 or int4. Two-word offsets would be a version 2, when needed.
- **Threads behind the same platform layer** (pthreads, Win32 threads). Optional: one thread is
  the default. A dense layer splits its outputs between threads and attention its heads, so
  every value is still one thread's sum in the same order: any thread count gives the same
  results, bit for bit. Threads spin for a few microseconds before sleeping, then sleep.
  The default count is the fast cores: a phone's big cores from
  `/sys/devices/system/cpu/*/cpu_capacity` (or their highest clocks), a Mac's performance or
  physical cores, at most 4.
- **Tokeniser and sampler in C89.** The tokeniser reads the model's own tokenizer.json: BPE
  (byte-level, as GPT-2, Llama 3, Qwen and SmolLM; SentencePiece-style with byte fallback, as
  Llama 2 and Mistral) and Unigram; Split, ByteLevel, Digits and Metaspace pre-tokenisers with
  a small regex engine and Unicode letter and number tables; added tokens. The sampler:
  temperature, top-k, top-p, min-p, repetition penalty, a seeded generator. A generator ties
  them to a session: the chat template's pieces, streaming, the cache kept across turns, no
  allocation per token. The Dart package exposes it (load, chat, stream, cancel).
- **Converter imports.** Hugging Face safetensors and config.json first, read with NumPy alone
  (no torch, no TensorFlow; jinja2 renders the chat template). GGUF later, for models
  published only that way.
- **Memory budget.** For a model with L layers, KV key/value heads of size D, context C and N
  weights at b bits:
  `memory ≈ N × b / 8 (mapped, file-backed, read every token) + C × L × 2 × KV × (D + 4) bytes
  (int8 cache, as it fills; D × 4 in float32) + arena (activations for one prompt chunk of
  32, about 2 MB)`. Only the last two are the process's own (private) memory; the system can
  drop and re-read the mapped weights. The chat path's context is 1024 by default (the
  model's maximum stays the converter's `--context`), and the tokeniser's table is read in
  place from the model file.
- **Size.** The Android library grows from 90 KB (arm64) to 151 KB with the tokeniser, the
  Unicode tables, the sampler, the generator, the platform layer and the kernels. The CI
  budget goes from 150 KB to 200 KB. The memory work of rayo#7 (int8 cache, float16 and
  3-bit kernels, the table reader) brings it to 171 KB (arm64) and 138 KB (armv7), within it.
- **Threads on phones.** The chat path's default is 2 of the fast cores on Android (at most
  4 elsewhere): on a low-end phone two cores use the bandwidth decoding needs.

## Numbers

**Memory, after rayo#7.** A chat of about 1000 tokens (SmolLM2-135M shape, random weights, a
synthetic tokeniser of its vocabulary's size; 2 threads): private memory (the process's own,
smaps_rollup's Anonymous) and total RSS at their peaks.

| SmolLM2-135M shape | File | Private | Total RSS |
| --- | --- | --- | --- |
| spike: q4 (float32 scales), float32 cache for 2048, tokenizer.json | 84 MB | 62 MB | 145 MB |
| q4, float32 cache (1024) | 80 MB | 46.5 MB | 125 MB |
| q4 (float16 scales), int8 cache, table in the file: the defaults | 80 MB | 15.1 MB | 94 MB |
| q4 and q3 (`--q3 mlp`), int8 cache | 70 MB | 15.1 MB | 84 MB |
| Qwen2.5-0.5B shape, q4, int8 cache | 282 MB | 13.4 MB | 297 MB |

The 40 MB target for a model's own memory on a 2 GB phone is met with the int8 cache (not
with the float32 one at 1000 tokens). In a memory cgroup of 64 MB or 48 MB, below the file's
size, the chat still finishes at about 9 tokens a second instead of 30: the system re-reads
the weights' pages; `MADV_WILLNEED` was as fast as no hint and faster than
`MADV_SEQUENTIAL`, so it is the default. Speed on the x86 host (noisy, shared): replies at
30 to 36 tokens a second on one thread for every combination; float16 scales cost about 15%
there (compute-bound), and are expected to help on phones (memory-bound): to be measured.
Quality: on synthetic matrices the dequantised error is the same with float16 scales, and
about twice as large with 3 bits (0.16 against 0.09 of the RMS); real models' quality is
not measured.

**Speed and memory of the spike (rayo#6):**

Measured on one 2.1 GHz Xeon (x86_64, AVX-VNNI), 4 cores, with models of the real shapes and
random weights (the real weights could not be downloaded from this environment; speed and
memory depend only on the shapes): 128 prompt tokens, then 64 decoded, the median of three
runs (the 0.5B shape: one run).

| Model, weights | File | Peak RSS | Prompt tokens/s, 1 / 2 / 4 threads | Decode tokens/s, 1 / 2 / 4 threads |
| --- | --- | --- | --- | --- |
| SmolLM2-135M, q4 | 80 MB | 95 MB | 129 / 175 / 211 | 41 / 74 / 52 |
| SmolLM2-135M, int8 | 129 MB | 146 MB | 258 / 309 / 350 | 51 / 81 / 56 |
| SmolLM2-135M, float32 | 513 MB | 528 MB | 46 / 127 / 202 | 18 / 28 / 38 |
| Qwen2.5-0.5B, q4 | 295 MB | 308 MB | 48 / 76 / 115 | 12 / 21 / 30 |
| Qwen2.5-0.5B, int8 | 473 MB | 491 MB | 82 / 135 / 225 | 18 / 28 / 32 |

- **Correctness.** rayo's logits match a NumPy reference of the Llama architecture (written
  from the definition, not from rayo's graph) within 2e-5 of the largest logit in float32,
  at every position and with the cache; with int8 and q4 the next token is the reference's at
  every position of the fixed prompts. Greedy tokens match NumPy exactly in float32. The
  tokeniser gives the tokenizers library's ids on five kinds of tokenizer.json.
- **Phones: estimates, to be replaced by measurements.** Under QEMU the 135M q4 shape runs on
  arm64 (SDOT) and armv7 (NEON) with a 93 MB peak: it works, but QEMU's times are not a
  phone's. Expected from memory bandwidth (decoding reads every weight once a token):

  | Device (estimate) | SmolLM2-135M q4 | Qwen2.5-0.5B q4 |
  | --- | --- | --- |
  | Low-end armv7 phone, 2 GB, 4 × Cortex-A53 at 1.4 GHz | 5 to 10 tokens/s, 100 MB | 1 to 3 tokens/s, 320 MB |
  | Mid-range arm64, 3 GB, 2 × Cortex-A76 (SDOT) | 20 to 40 tokens/s | 6 to 12 tokens/s |
  | Recent arm64, 8 GB, Cortex-X class | 50 to 90 tokens/s | 15 to 30 tokens/s |

## Consequences

- The app can offer an on-device assistant on every phone it supports, with no Google
  library, in the F-Droid build too, and with the same engine and tests as the Ask models.
- rayo grows by about 60 KB and by a platform layer; its C89 and no-allocation rules hold.
  The system headers stay in `src/platform/`, as the intrinsics stay in `src/kernels/`.
- The converter becomes the place where models differ. A new architecture is a converter
  change plus, at most, a new general op.
- Models are downloaded by the app (they are tens to hundreds of MB), checked by SHA-256 and
  mapped from the app's files directory; they are not in the APK.

## Rollout

1. **Spike (rayo#6)**: the ops, int4 kernels, mapping, threads, tokeniser, sampler, generator,
   the converter from Hugging Face, the chat example and the Dart API; CI on every platform.
2. **Real models**: convert SmolLM2-135M/360M-Instruct and Qwen2.5-0.5B-Instruct where Hugging
   Face can be reached; compare the first greedy tokens with the transformers library;
   measure on a low-end armv7 phone and a recent arm64 one, and replace the estimates above.
3. **Memory** ([rayo#7](https://github.com/pesaclarity/rayo/pull/7)): the int8 key/value
   cache; the session's own context (1024 by default) and prompt chunks of 32; the
   tokeniser's table written by the converter into the model file and read in place
   (reading Qwen's tokenizer.json took about 70 MB for a moment); float16 group scales (4.5
   bits a weight); optional 3-bit groups by role; madvise hints and a memory report that
   splits private from file-backed memory. Numbers above.
4. **Speed**: ARMv8.6 i8mm and AVX-VNNI kernels for int4, more rows per block for prompts,
   pinning threads to the big cores.
5. **The app**: a separate decision on what the assistant does, which model ships, how it is
   downloaded, and its licence notices.

## Risks and open questions

- **No phone measured yet.** The table above is estimates until step 2.
- **Quality at 4 bits** on 135M-parameter models: measure the real models' agreement with
  float32 before choosing q4 over int8 for each.
- **Thermal and battery**: sustained decoding on big cores heats phones; the defaults (at most
  4 threads, short replies) may need lowering after measurement.
- **Tokeniser coverage**: text is taken to be NFC already, which holds for most input; new
  tokenizer.json features are refused by name until added.
- **Llama's licence** stays out unless decided separately.

# 0020. rayo: our own inference engine in C89 replaces TFLite

Date: 2026-11-17 · Status: Proposed · Amends [0019](0019-on-device-ml-without-google.md)

## Context

[0019](0019-on-device-ml-without-google.md) put today's small models in pure Dart and kept a native runtime, built from source, for models too big or slow for Dart. Since then:

- The owner wants Google's TFLite/LiteRT out of both builds, and the models on a fast native runtime of our own, not on Dart.
- Pure Dart takes 5 to 7 ms for one Ask question on a 2.1 GHz Xeon core. TFLite takes 0.06 to 0.08 ms with XNNPACK and 0.25 to 0.32 ms with its built-in kernels.
- `tflite_flutter` ships Google's prebuilt LiteRT AARs (`libtensorflowlite_jni.so`, about 4.3 MB on arm64) in both builds. F-Droid accepts no prebuilt native libraries ([0006](0006-fdroid-build-without-prebuilt-libraries.md)).

The models are the Ask reader (an int8 CNN, 240k parameters) and the Ask writer (an int8 three-layer transformer, 532k parameters, decoding word by word). Later come the message reader (the reader's family) and the categoriser, which also learns on the phone.

## Options

- **(a) ggml**: MIT, the C library behind llama.cpp. Tried first: the reader ran on it, 30 of 30 labels, 0.5 ms per question (0.11 ms on the question's real tokens only), 1.45 MB on arm64. Rejected:
  - Its CPU backend, GGUF reader and scheduler are C++, so the app would carry C++ and 4 MB of vendored code.
  - Its general kernels are slow at this size.
  - Matching TFLite's int8 arithmetic needed custom ops anyway.
- **(b) ncnn, MNN, ONNX Runtime**: C++ and CMake. Each is larger than our needs, and none runs TFLite's dynamic-range int8 the way TFLite does.
- **(c) Our own engine for our ops**: small, plain C, built by the Dart build hook with the platform's compiler. We own every op; each one is tested against NumPy and TFLite.
- **(d) Pure Dart (0019)**: no native code, but 20 to 80 times slower than TFLite.

## Decision

**rayo** (Spanish for lightning) is our inference engine, in its own repository ([pesaclarity/rayo](https://github.com/pesaclarity/rayo), AGPL-3.0). It replaces TFLite for every model in both builds.

- **C89 (ANSI C)**:
  - Every file builds with gcc and clang as `-std=c89 -pedantic -Werror -Wall -Wextra -Wconversion -Wshadow -Wdeclaration-after-statement -Wvla`.
  - Fixed-size types (`rayo_i8` to `rayo_u32`) are defined with compile-time size checks. There are no 64-bit integers: dimensions are 32-bit, and every tensor is below 2^31 bytes.
  - rayo has its own bounded formatting, and computes exp, sqrt and erf in double, since C89 lacks them.
  - Compiler extensions (SIMD intrinsics, target attributes, CPU checks) live only in the kernel files for x86 and NEON. No C++, no other library.
- **Built from source**:
  - A Dart build hook (`native_toolchain_c`) compiles `src/` for Android (arm64-v8a, armeabi-v7a, x86_64), iOS, macOS and Linux.
  - Nothing is downloaded or prebuilt, so F-Droid builds it offline like the rest of the app.
  - The stripped Android library is 63 KB (arm64-v8a), 49 KB (armeabi-v7a) and 66 KB (x86_64).
- **TFLite's numbers**: int8 dense and conv1d layers run as TFLite's dynamic-range ("hybrid") kernels do:
  - each layer's input is quantised to int8 on the fly, per row, or per tensor and dilation phase for a convolution;
  - products are exact in int32;
  - dequantisation is the same as TFLite's.
- **Fast where it counts**:
  - The int8 products have a generic C version and SIMD ones: SSE4.1/AVX2 and NEON at compile time, AVX-VNNI and ARM SDOT chosen per session at run time.
  - Every SIMD kernel is checked against the generic one in the tests.
  - A transformer decodes with a key/value cache, instead of recomputing all 64 positions per word as the app's TFLite path does.
- **Memory planned once**:
  - A session plans every activation into one arena; buffers whose lifetimes do not overlap share memory.
  - A run allocates nothing. The tests count this.
  - Model files are untrusted: every size and offset is checked, and damaged files fail cleanly under ASan and UBSan.
- **Model format**:
  - The plain-weights file pesaclarity-ml already writes (PCWT version 1, `docs/WEIGHTS.md`), with the graph in an optional header field.
  - Readers ignore fields they do not know, so one `ask-reader.weights` serves rayo and the Dart reader.
  - pesaclarity-ml reads the weights out of the released `.tflite` files, so they are the int8 values TFLite runs.
  - Weights stay under CC BY-SA 4.0.
- **Fallbacks stay**:
  - When the native library cannot load (an ABI rayo was not built for), the reader falls back to the pure-Dart reader, then to the rules.
  - Replies fall back to the templates.
- **CI**: rayo has GitHub Actions CI, unlike the app and ML repositories, which check before each push:
  - Linux x86_64 with gcc and clang;
  - the sanitizers;
  - native Linux arm64, where NEON and SDOT run;
  - armv7 under QEMU;
  - macOS;
  - the Android NDK builds, with a size budget;
  - the Dart package.

## Numbers

All measured on one core of a 2.1 GHz Xeon. No phone has been measured yet.

| | Ask reader, per question | Ask writer, per reply (about 13 steps) |
| --- | --- | --- |
| rayo, AVX-VNNI | 0.15 to 0.19 ms | 0.84 to 0.89 ms |
| rayo, AVX2 | 0.34 ms | |
| rayo, generic C | 0.80 ms | |
| TFLite, XNNPACK, 1 thread | 0.08 ms | 4.1 ms |
| TFLite, built-in kernels | 0.28 ms | 39 ms |
| pure Dart (0019) | 5 to 7 ms | |

- **Reader**: every intent and slot label is TFLite's on the 30 recorded questions, with probabilities within 4.3e-5.
  - It runs all 96 positions, because TFLite quantises a convolution's whole input; cutting the padding would change TFLite's numbers.
  - The gap to XNNPACK (about 2.4×) is mostly XNNPACK's 512-bit VNNI and packed weight layouts. That is future work.
- **Writer**: the next-token choice is TFLite's at every one of the 2243 steps of the 172 recorded replies. All 265 seeded replies the app's tests ask for come out token for token as TFLite recorded them, and every reply check passes.
  - Its logits are within 0.1 to 0.15 of TFLite's: the size of TFLite's own XNNPACK and built-in kernels' disagreement (0.145), from int8 rounding.
  - rayo is about 5 times faster than the app's TFLite path, because of the key/value cache.
- **Memory**: the reader needs 251 KB of weights and a 241 KB arena; the writer 592 KB and 435 KB.

## Consequences

- Neither build ships a Google ML library once the app switches over:
  - `tflite_flutter`, the LiteRT AARs and the GPU-library exclude in Gradle go;
  - `tool/prepare_foss.sh` no longer needs to touch them.
  - The Play build keeps Google sign-in, a separate decision.
- **We own every op.** A new layer type is a rayo op with a generic kernel, a NumPy reference and tests, and a SIMD kernel when it pays.
  - The categoriser's learning on the phone needs a backward pass for its two dense layers. That is not in rayo yet, so the categoriser stays on the rules until it is.
- **Results are TFLite's, to TFLite's own precision.** Bit-exact agreement is not possible: TFLite's two kernel sets disagree with each other by the same amount.
- rayo's version is pinned in the app like the parser packs: a git dependency on a release tag.

## Risks and open questions

- **Phones**: NEON and SDOT are checked on CI's arm64 runners and under QEMU, but nothing has been timed on a phone. Measure on a low-end 32-bit phone and a recent arm64 one before the switch-over ships.
- **iOS**: built by the same hook, but not yet built or run.
- **F-Droid**: confirm that a Dart build hook compiling C with the NDK is accepted in the recipe, as SQLite's already is.
- **Bigger models**: rayo's ops cover today's transformer. A larger language model would need more kernels (more blocking, threads, float16 weights). Each would be measured before it is added.

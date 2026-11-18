# 0022. Our own language model for Ask

Date: 2026-11-18 · Status: Proposed · Builds on [0017](0017-ask-on-the-phone.md) and [0021](0021-small-language-models-on-rayo.md)

This is a plan. Nothing here has been trained. Every number marked *estimate* is arithmetic or
a judgement, not a measurement, and is replaced by a measurement at the stage that can make it.

## Context

Ask ([0017](0017-ask-on-the-phone.md)) answers with two tiny models and the app's own lookups
(pesaclarity-ml `docs/ASK.md`, schema version 1):

- **Reader**: a CNN of 239,589 parameters (272 KB TFLite, about 251 KB of plain weights) that
  picks one of 30 intents and tags 7 slot labels per token (category, payee, period, item,
  account, amount). The app fuzzy-matches the slots against the space's records. v3 scores:
  conformance questions (68) intent 1.000, all slots exact 0.971; misses (62) 0.984 / 0.968;
  hard held-out (2,000) intent 0.964, payee/account exact 0.926; app questions (65) intent
  1.000, names 0.959. All on synthetic or hand-written questions.
- **Writer**: a decoder of 3 attention layers (width 128, 4 heads, feed-forward 256, 473-token
  vocabulary, 64-token window, about 590 KB of weights) that writes words and placeholders
  from fact tokens. A checker refuses a digit, a placeholder not given, a repeat or a missing
  required one; the app then uses the kind's fixed sentence.

Both run on rayo ([0020](0020-rayo-our-own-inference-engine.md)) and ship in the APK. What they
cannot do: questions outside the templates' shapes, Swahili and Sheng beyond a few
templates, follow-ups other than `refine`, and explanations ("why is food up?").
[0021](0021-small-language-models-on-rayo.md) (Proposed) gives rayo a generic Llama-family
decoder from config.json and safetensors, 4-bit weights, mapped files, threads, a tokeniser and
a sampler. Its measured SmolLM2-135M q4 file is 80 MB with a 95 MB peak, too large for a 2 GB
phone next to the app, and SmolLM2's tokeniser is English-first.

Rules that bind this plan:

- **Synthetic only**: no real user data in training or evaluation, ever.
- **No Google**: no Google-made models (Gemma, T5/mT5, ByT5), datasets (C4, mC4 and anything
  derived from it, such as MADLAD-400 or WURA), libraries (TensorFlow, JAX, SentencePiece) or
  services (Colab, TPUs, Google Translate) in the new model's pipeline.
- **Size**: release APKs are checked to be under 25 MB per chip (pesaclarity-app README; about
  19 MB for 64-bit `play` today), so the model cannot ship in the APK.
- **Network**: the privacy policy (version 2.1) states each request the app makes. Ask's
  section says "Nothing is sent anywhere" and that its models "ship inside the app". A model
  download is a new request and changes that text.
- **New screens and cards need approved mockups before they are built.**

## Decision (proposed)

**Train our own decoder-only model of about 57M parameters ("Ask LM") from scratch, in
PyTorch, on open-licensed text and synthetic task data. It writes a structured query, never
an amount; the app runs the query on the space's database; the model then writes the reply
from the returned figures, copying every number. rayo runs it with grammar-constrained
decoding. It is an opt-in, signed download; without it Ask keeps today's reader and writer.**

### What it does

- Free-form money questions in English, Swahili and Sheng, follow-ups across turns ("na
  mwezi uliopita?", "only Naivas", "why?"), and short explanations from the figures it is given.
  No financial advice; out-of-scope questions get a refusal, as `out_of_scope` today.
- **Two steps per turn, one model:**
  1. **Query.** From the conversation so far, the model writes one query in a small grammar
     over the app's own data. The app parses it, runs it on the open space's database with the
     same providers the screens use, and formats the results as the screens do ("KES 4,250").
  2. **Reply.** The results go back into the context; the model writes a short reply.
- **The model never invents an amount.**
  - In the query, a number (an amount filter such as "over 1,000") can only be one the question
    contains: the grammar offers only those.
  - In the reply, digits are allowed only as an exact copy of a value in the results: the
    grammar's copy rule offers those strings and nothing else. Number words ("thousand",
    "elfu", "laki", "milioni") are refused outside copied values.
  - The app's checker then re-checks every reply: each digit run is part of a copied result
    value; names, dates and periods are ones the results or the space contain; length and
    stray tokens as in today's checker. On any failure, or if the query does not run, the turn
    falls back to the reader and writer (or the writer's fixed sentence for those results).
- **Follow-ups**: the context holds earlier turns as their queries and replies, not raw
  results; each turn's query is complete, so the tags above the box (0017) are read from it.
  1,024 tokens of context, enough for a fixed prefix and about three turns (*estimate*).

### The query grammar

A small text grammar, versioned with `ask_schema_version`, kept in pesaclarity-ml next to
`docs/ASK.md`. Sketch (final names in the schema):

```text
query   := intent (" " filter)* | "clarify" | "out_of_scope" | "smalltalk"
intent  := "spent" | "received" | "sent_to" | ... the 24 money intents of schema v1
filter  := "category:" CATEGORY | "payee:" PAYEE | "account:" ACCOUNT | "item:" ITEM
         | "period:" PERIOD | "compare:" PERIOD | "amount" ("<" | ">") QNUMBER
         | "by:" ("payee" | "category" | "month") | "top:" SMALLINT
PERIOD  := "today" | "this_week" | "last_week" | "this_month" | "last_month" | "this_year"
         | "last_year" | "month:" YYYY "-" MM | "since:" DATE | "range:" DATE ".." DATE
CATEGORY, PAYEE, ACCOUNT, ITEM := one of the open space's own names, given at run time
QNUMBER := one of the numbers in the question, given at run time
```

Names are closed sets: the model can only pick a payee the space has, so the fuzzy matching
moves into decoding (the app passes each set with its aliases and the shortcuts people taught
Ask). Unknown names lead to `clarify`.

### Constrained decoding in rayo, generic

A new rayo feature, not model code: a grammar given as data (an EBNF-like text compiled at load
time into a bounded automaton) plus run-time string sets (the names, the question's numbers,
the results' values). At each step rayo masks the logits of tokens the automaton cannot
accept, by walking a trie of the vocabulary built once per tokeniser. C89, no allocation per
token (the automaton's stack and the sets live in the session's arena), any model and any
grammar; tested against a Python reference on random grammars. The Dart package passes the
grammar and the sets per call.

### Architecture

Llama-style, so rayo-convert reads it as the Llama family (0021) and rayo needs no model code:
pre-norm RMSNorm (eps 1e-5), RoPE (theta 10,000, no scaling, 1,024 positions), grouped-query
attention with 8 query heads and 2 key/value heads, SwiGLU, no biases, tied input and output
embeddings, `LlamaForCausalLM` config.json and safetensors. Own byte-level BPE tokeniser of
16,384 tokens (below).

| Config | d_model | Layers | Heads (q / kv), head size | SwiGLU hidden | Parameters |
| --- | --- | --- | --- | --- | --- |
| **A (proposed)** | 512 | 16 | 8 / 2, 64 | 1,536 | 56.64M |
| B | 576 | 14 | 8 / 2, 72 | 1,536 | 58.22M |
| B16 | 576 | 16 | 8 / 2, 72 | 1,536 | 65.19M |
| Small (alternative) | 384 | 16 | 8 / 2, 48 | 1,024 | 31.08M |

Parameters of A, by part:

| Part | Formula | Parameters |
| --- | --- | --- |
| q and o projections, per layer | 2 × 512 × 512 | 524,288 |
| k and v projections, per layer | 2 × 512 × (2 × 64) | 131,072 |
| SwiGLU (gate, up, down), per layer | 3 × 512 × 1,536 | 2,359,296 |
| two RMSNorms, per layer | 2 × 512 | 1,024 |
| **one layer** | | **3,015,680** |
| 16 layers + final norm | | 48,251,392 |
| tied embedding / output head | 16,384 × 512 | 8,388,608 |
| **total** | | **56,640,000** |

A is chosen over B for head size 64 (rayo's and most kernels' common case) and a file under
36 MB with int8 embeddings. B fits only with 4-bit embeddings; B16 does not fit.

### Size and memory (A), arithmetic

Weights in 4-bit groups of 32 with a float16 scale each: 4 + 16/32 = **4.5 bits a weight**.
This needs rayo's float16 group scales (0021 rollout step 3); with today's float32 scales it
is 5 bits and A would be 38.6 MB. MB here is 10^6 bytes.

| Item | Formula | Size |
| --- | --- | --- |
| Layer weights, 4-bit | 48.25M × 4.5 / 8 | 27.14 MB |
| Embedding, int8 per row + float16 row scale | 8.39M + 16,384 × 2 | 8.42 MB |
| Norms, float32 | (16 × 1,024 + 512) × 4 | 0.07 MB |
| **File, int8 embeddings** | | **35.6 MB** (≤ 36 MB) |
| File, 4-bit embeddings (if quality holds) | 27.14 + 4.72 + 0.07 | 31.9 MB |
| KV cache, int8, 1,024 tokens | 16 layers × 1,024 × (2 × 2 × 64 bytes + 2 × 2 × 2 bytes of float16 scales) | 4.33 MB |
| KV cache in float32, for comparison | 16 × 1,024 × 2 × 2 × 64 × 4 | 16.8 MB |

| Memory (*estimate*) | Size |
| --- | --- |
| Mapped weights (file-backed, clean, droppable under pressure; all read each token) | 35.6 MB |
| KV cache, int8, full 1,024 tokens | 4.3 MB |
| Arena: one 64-token prompt chunk (SwiGLU 64 × 1,536 × 4 × 2, scores 8 × 64 × 1,024 × 4, activations, logits) | 3 to 4 MB |
| Tokeniser in a compact table (needs 0021 step 3; tokenizer.json is not parsed on the phone) | under 1 MB |
| Grammar automaton, vocabulary trie and run-time sets | under 1 MB |
| Sampler, generator, library code | under 0.5 MB |
| **Private memory** | **about 10 MB** (target ≤ 15 MB) |
| **Total RSS added by Ask LM** | **about 46 MB** (target ≤ 50 MB) |

### Speed (*estimates*, not measured)

Decoding reads every weight once a token, so it scales with file size. Scaled from 0021's
estimate for SmolLM2-135M q4 (5 to 10 tokens/s on a 4 × Cortex-A53 armv7 phone; its file is
80 MB, A's 35.6 MB), and from published llama.cpp runs on Cortex-A53 boards:

| Device (*estimate*) | Decode tokens/s | Prompt tokens/s |
| --- | --- | --- |
| 2 GB armv7, 4 × Cortex-A53 at 1.4 GHz (NEON, no SDOT) | 12 to 25 | 40 to 100 |
| 2 to 3 GB arm64, Cortex-A55 cluster (SDOT), 4 threads | 25 to 50 | 100 to 250 |

A turn (*estimate*): the fixed prefix (about 120 tokens) runs once per app session and stays in
the cache; then about 40 new prompt tokens, a query of about 20 tokens, about 60 tokens of
results and a reply of about 40, streamed. First word of the reply after about 2 to 4 s on
the A53 phone and 1 to 2 s on the A55 one; the whole reply 2 to 3 s later on the A53.

### Tokeniser

Byte-level BPE with 16,384 tokens, trained with Hugging Face `tokenizers` (Apache-2.0; not
SentencePiece) on a balanced sample: about 40% English, 40% Swahili and Sheng, 20% money text
(synthetic M-PESA and bank messages in the formats of ujumbe-parsers' fixtures, amounts,
dates, account masks). Digits split one by one (the Digits pre-tokeniser, which rayo reads) so
copied amounts tokenise the same everywhere. Special tokens for the turn structure (system,
user, query, results, reply, end). Measured before pretraining: tokens per word on held-out
English, Swahili and Sheng text, against SmolLM2's and Qwen2.5's tokenisers.

### Data

No real user data. No Google-made data. Each source is recorded in a data card with its
licence, version, hash and attribution; a source with a non-commercial (NC), no-derivatives
(ND) or unclear licence is left out.

| Source | Licence | Use |
| --- | --- | --- |
| English Wikipedia | CC BY-SA 4.0 (attribution, share-alike) | pretraining |
| Swahili Wikipedia | CC BY-SA 4.0 | pretraining |
| FineWeb-Edu (Hugging Face, from Common Crawl) | ODC-By 1.0, and Common Crawl's terms of use | pretraining, English |
| FineWeb-2, Swahili subset (Hugging Face) | ODC-By 1.0, and Common Crawl's terms of use | pretraining, Swahili |
| Masakhane and other African NLP datasets | per dataset, checked one by one | mostly evaluation of Swahili; pretraining only where CC BY, CC BY-SA, CC0 or ODC-By |
| ujumbe-parsers fixtures (synthetic messages: fake names, masked fake numbers) | Apache-2.0 | message-text templates |
| akili's Ask templates, names and replies (`questions.py`, `names.py`, `replies.py`) | ours (AGPL-3.0 code) | task data |
| Teacher outputs (below) | the teacher's licence; we credit it | paraphrases, Swahili/Sheng variants, explanations |

- **Teachers**: Qwen2.5-1.5B-Instruct and Qwen2.5-7B-Instruct (Apache-2.0) and
  SmolLM2-1.7B-Instruct (Apache-2.0). Not Qwen2.5-3B (Qwen's research licence), not Llama
  (community licence, 0021), not Gemma. Teachers see only synthetic scenarios and templates.
  Every teacher-written example passes the same checks as the model's replies (numbers copied,
  names given) or is dropped; Swahili and Sheng examples are sampled for review by speakers.
- **Pretraining mix, 4B tokens** (*estimate*; unique Swahili volume to be measured, repeated
  up to 4 epochs if short): FineWeb-Edu 65%, English Wikipedia 10%, Swahili (Wikipedia and
  FineWeb-2) 15%, synthetic money text in all three languages 10%, weighted towards the end.
- **Task data (SFT)**: a scenario generator makes synthetic spaces (accounts, payees,
  categories, items, transactions, invented amounts, names from `names.py`), dialogues of one
  to four turns over them in English, Swahili, Sheng and mixed, the reference query for each
  turn, the results from a Python reference executor of the grammar (tested against the app's
  Dart executor on shared fixtures), and the reply. About 300k to 500k dialogues, 100M to
  150M tokens (*estimate*), with clarify, out-of-scope, small talk and no-data turns.
- **Decontamination**: before any training, drop every example whose normalised text equals,
  or shares an 8-word sequence with, the 68 conformance questions, the 62 misses, the 65 app
  questions, the 45 hard held-out templates, the new Swahili/Sheng and dialogue test sets; keep
  `names.HELD_OUT` words out of all training text. The script reports what it removed; teacher
  prompts never contain test items.

### Training recipe

PyTorch (BSD) only, no TensorFlow or JAX; our own short training loop or `transformers`'
Llama classes (Apache-2.0), bfloat16, AdamW, warmup then cosine decay, sequence length 1,024.
Code in pesaclarity-ml under AGPL-3.0, seeded and reproducible.

| Stage | What | Compute (*estimate*, 6 × N × D) |
| --- | --- | --- |
| 0. Tokeniser | BPE as above | CPU, under an hour |
| 1. Pilots | 2 to 3 runs of the Small config on 0.3B tokens: mix, learning rate, tokeniser check | about 3 GPU-hours |
| 2. Teacher data | about 30M to 50M generated tokens from the 7B teacher | 4 to 8 GPU-hours |
| 3. Pretraining | A on 4B tokens (about 70 tokens a parameter; Chinchilla's 20 would be 1.13B): 6 × 56.6M × 4B = 1.36 × 10^18 FLOPs | 3 to 4 h on one A100 at 30 to 40% utilisation (95 to 125 TFLOP/s of 312), about 1.5 h on an H100; 5 to 8 h with evaluation and restarts |
| 4. SFT | task dialogues, loss on query and reply tokens, 2 to 3 epochs (about 0.4B tokens) | about 1 GPU-hour |
| 5. Preference | rejection sampling and DPO: pairs scored by the reference executor and checker (right query and copied numbers against wrong period, swapped figure, invented name, answering out of scope) | 1 to 2 GPU-hours |
| 6. QAT | fake 4-bit quantisation exactly as rayo stores it (groups of 32, float16 scale, largest magnitude to −8), straight-through estimator, int8 embedding, over the last part of SFT and preference | 1 to 2 GPU-hours |
| 7. Export | safetensors + config.json + tokenizer.json, rayo-convert to int4, gates run on rayo's output | CPU |

Chinchilla's 1.2B tokens for 57M parameters would be 4.1 × 10^17 FLOPs, about 1.1 A100-hours at
100 TFLOP/s; over-training to 3B to 5B tokens costs 3 to 5 hours more and is cheap at this size.

**Where it runs.** This cloud container has no GPU and little disk. The plan needs one rented
A100 80 GB or H100 for about 20 to 30 GPU-hours, 40 to 60 with reruns (*estimate*), about
USD 60 to 180 at USD 1.5 to 3 an hour (*estimate*; prices vary by provider and month), plus
about 100 GB of disk for data shards. The owner decides the provider; data is downloaded there,
not here.

### Evaluation gates

All gates run on the 4-bit file under rayo, on CPU, not on the PyTorch model. The reader and
writer are scored on the same sets.

| Gate | Set | Pass |
| --- | --- | --- |
| **Amount exactness (hard)** | held-out synthetic dialogues, at least 5,000 turns in three languages, unseen templates and names | 100% of numbers in replies copied from that turn's results, with the app's checker off; any miss blocks release |
| Right figure | same set: the executed query's result equals the reference query's | at least 97% English, 93% Swahili and Sheng (*proposed*, set after the first run) |
| Conformance | the 68 questions of `ask_examples.v1.json`, intent and slots read from the query | intent 1.000, all slots exact ≥ 0.971 (reader v3) |
| Misses, hard held-out, app questions | the 62, 2,000 and 65 sets | at least reader v3: 0.984 / 0.968; 0.964 and 0.926; 1.000 and 0.959 |
| Swahili and Sheng | new hand-written sets, at least 200 questions each, reviewed by speakers | thresholds set after the first review |
| Follow-ups | multi-turn sets (change period, payee, "why?") | at least the reader's `refine` handling, scored per turn |
| Refusals | out-of-scope and advice questions; in-scope look-alikes | ≥ 98% refused; ≤ 2% in-scope wrongly refused |
| Reply quality | 300 replies rated by reviewers against the writer's | not worse, by reviewers' rating |
| Phones | a 2 GB armv7 Cortex-A53 phone, a 2 to 3 GB arm64 Cortex-A55 phone, one de-Googled phone (no Google services) | first reply word ≤ 5 s (A53) and ≤ 3 s (A55); decode ≥ 10 tokens/s on the A53; private memory ≤ 15 MB, total ≤ 50 MB; no throttling over 20 questions |
| Size | the file and the app | file ≤ 36 MB; APKs still under 25 MB; rayo inside its CI size budget |

### Shipping

- **Download, opt-in.** Offered in Ask (a card) and Ask settings; Wi-Fi (unmetered) only by
  default; resumable (HTTP range requests into a partial file); size and SHA-256 pinned in a
  manifest signed with Ed25519 against a key pinned in the app, as ujumbe packs' index is;
  the manifest names the rayo and `ask_schema_version` it needs. The file is checked before
  it is mapped and kept in the app's files directory, out of backups, and can be deleted. The
  request carries no account, device or install id.
- **Where from**: a GitHub release of pesaclarity-ml, as ujumbe packs are fetched from GitHub
  releases, or our own server (which the policy says Google Cloud runs). The owner decides.
- **Ask keeps working without it**: the reader and writer stay in the APK and are the
  fallback for every turn.
- **Mockups first**: the offer card, the settings row (size, progress, pause, delete, version)
  and any change to Ask's reply view need approved mockups before they are built.
- **Privacy policy**: a version bump (from 2.1) and changelog entry: the Ask section says the
  optional model is downloaded once, from where, what the request carries, that the host may
  keep standard request logs, and that questions and answers still never leave the phone.
- **F-Droid**: the weights and every training source must be free (above), and the training
  code and data card public, so the model is rebuildable in principle. To confirm with F-Droid
  before enabling the download there: whether an opt-in download of free weights from our
  release needs an anti-feature (NonFreeNet, NonFreeAssets) or none. Until confirmed, the
  `foss` build does not offer the download.
- **The model's licence**: our weights under CC BY-SA 4.0, as app-track weights are
  (pesaclarity-ml `LICENSE-WEIGHTS`), which also meets Wikipedia's share-alike if it applies to
  weights; a NOTICE and model card credit Wikipedia, FineWeb-Edu and FineWeb-2 (ODC-By), the
  teachers (Apache-2.0) and ujumbe-parsers (Apache-2.0), and state the limits and the scores.

## Consequences

- Ask can take free-form questions in three languages, follow-ups and short explanations, on
  phones without Google services, with numbers that come only from the database.
- rayo gains a generic grammar-constrained sampler and needs 0021's float16 scales, int8
  key/value cache and compact tokeniser table before this ships.
- pesaclarity-ml gains a PyTorch pipeline next to its Keras one, a scenario generator, a
  reference executor that must match the app's, and a data card to keep current.
- The app gains a query executor over the grammar, a download with its screens, a stricter
  checker and a policy change. Two Ask paths exist until the reader and writer are retired,
  if ever.
- Money is spent on GPU time; the amount is the owner's call.

## Stages

1. **Accept this record**, with the owner's answers below.
2. **rayo**: float16 group scales, int8 key/value cache, compact tokeniser table (0021 step 3);
   grammar-constrained decoding with run-time sets. Measure 0021's real models on phones.
3. **Grammar and data** (CPU, no GPU): the grammar in `ask_schema_version` 2, the reference
   executor and its fixtures shared with the app, the scenario generator, decontamination, the
   new Swahili/Sheng and dialogue test sets with speakers' review, the tokeniser.
4. **GPU**: pilots, teacher data, pretraining, SFT, preference, QAT, export.
5. **Gates** on rayo, then on the three phones.
6. **App**: mockups, executor, checker, download, policy bump, behind a setting; Play first,
   F-Droid after its answer.

## Alternatives

| Option | For | Against |
| --- | --- | --- |
| Fine-tune SmolLM2-135M (Apache-2.0) | proven pretraining, no pretraining cost | 135M parameters: 80 MB file and 95 MB peak measured at q4 (0021), over the 36 MB and 50 MB targets; 49,152-token English-first tokeniser, weak Swahili and Sheng |
| Small config, about 31M | 20.3 MB file (int8 embeddings), 17.5 MB with 4-bit; KV 3.3 MB; about twice as fast (*estimate*) | weaker Swahili and explanations; worth a pilot, and the fallback if A misses the phone gates |
| Query-only model: the LM writes queries, today's writer replies | smaller reply risk, no copy rule in replies | no explanations, no Swahili or Sheng replies |
| Keep the reader and writer | no cost, already shipped, 100% offline in the APK | no free-form questions, few languages, no explanations |

## Risks and open questions

- **Swahili and Sheng**: open Swahili text is much smaller than English and open Sheng text
  is scarce; Sheng rests on synthetic data and speakers' review. Quality may stay behind English.
- **Phone speed**: all speed numbers are estimates until stage 5; the A53 armv7 phone is the
  likely limit. Sustained decoding heats phones.
- **Explanations at 57M parameters** may be shallow or wrong in reasoning even with right
  numbers; the gates judge wording only by reviewers.
- **Licences**: whether share-alike or ODC-By attach to trained weights is unsettled; our
  choice of CC BY-SA 4.0 is meant to meet either reading. Each Masakhane dataset's licence is
  to be checked before use.
- **F-Droid's answer** on downloaded weights.

Questions for the owner:

1. **GPU budget and provider**: about 20 to 60 GPU-hours, USD 60 to 180 (*estimate*); which
   provider, and is that budget approved?
2. **Languages**: priority between English, Swahili and Sheng for the first release, and
   whether Swahili/Sheng gates block it or follow.
3. **Reviewers**: who reviews Swahili and Sheng test sets, teacher data and replies, and how
   many hours they have.
4. **Hosting**: GitHub releases or our server for the download.
5. **Config**: A (57M) as proposed, or the 31M pilot first?

# 0023. pesaclarity-ml on PyTorch, without TensorFlow

Date: 2026-11-18 · Status: Proposed · Builds on [0020](0020-rayo-our-own-inference-engine.md); goes with [0022](0022-own-ask-language-model.md)

This is a plan. Figures marked *estimate* are judgements, not measurements.

## Context

The owner prefers PyTorch. pesaclarity-ml (akili, `origin/dev`) trains every model with Keras
on TensorFlow, exports TFLite, and has rayo-convert read the TFLite file into the PCWT weights
the app runs ([0020](0020-rayo-our-own-inference-engine.md)). The app no longer contains TFLite:
rayo runs the Ask models, and the categoriser has no runtime yet (its opener is null). TFLite
survives only as the reference that akili, rayo's converter tests and the app's recorded decodes
are checked against. Ask LM ([0022](0022-own-ask-language-model.md), PR #71) is PyTorch from
the start, so moving the small models leaves one framework. The privacy accountant is Google's
`dp-accounting`, against the no-Google rule ([0019](0019-on-device-ml-without-google.md), [0022](0022-own-ask-language-model.md)).

### Inventory (pesaclarity-ml `origin/dev`)

| File or module | What uses TensorFlow, Keras or Google | Replacement |
| --- | --- | --- |
| `pyproject.toml` | `[train]` extra: `tensorflow>=2.18,<2.22` (Keras 3 comes with it), `mlflow`; core: `dp-accounting>=0.6.0,<0.7`; `[fl]`: `flwr`; mypy overrides for `tensorflow.*`, `keras.*`, `dp_accounting.*` (and `tf.function` decorators in `train.dp_sgd` and `federated.keras_trainer`); pytest marker `tf`; description says "TFLite export" | `[train]`: `torch` (CPU wheels), `onnx`, `mlflow`; core: `opacus` in place of `dp-accounting`; `flwr` unchanged; marker `torch` |
| `src/akili/model.py` | categoriser: Keras embeddings (hashed payee tokens, 5 context columns), masked mean, `hidden` and `category` dense; about 70k parameters, about 80 KB TFLite, 512 KB budget | `torch.nn` module with the same layer names |
| `src/akili/train/pipeline.py` | categoriser training (`keras.optimizers.Adam`), `export_tflite`, `val.accuracy` | torch loop; export via ONNX to rayo |
| `src/akili/train/dp_sgd.py` | custom DP-SGD (tensorflow-privacy does not support Python 3.12 or Keras 3): per-example `GradientTape`, `tf.clip_by_global_norm`, `tf.vectorized_map`, Poisson sampling, per-person unit | torch: per-person gradients with `torch.func` (`vmap` of `grad`), the same clipping, noise and sampling; Opacus's accountant |
| `src/akili/privacy/accounting.py`, `ledger.py` | `dp_accounting` RDP accountant (repo's own orders) for DP-SGD and DP-FedAvg; PLD accountant for `epsilon_pld`; the ledger composes runs with RDP | Opacus `RDPAccountant` with the same orders; `PRVAccountant` for the tighter figure |
| `src/akili/privacy/mia.py` | scores the exported TFLite file (`run_tflite`) and shadow models converted to TFLite | score the shipped rayo file (`rayo_convert.runtime`); shadows the same way |
| `src/akili/privacy/audit.py`, `gate.py` | NumPy only | unchanged (the gate reads the new accountant's figures) |
| `src/akili/federated/keras_trainer.py` | local training with `tf.GradientTape`, `keras.optimizers.SGD` | `torch_trainer.py` |
| `src/akili/federated/client.py`, `updates.py`, `simulation.py`, `flower.py`, `run.py` | NumPy and Flower (`NumPyClient`, `ClientApp`); `run.py` imports the Keras trainer and `export_tflite` | stay; `run.py` takes the torch trainer and rayo export |
| `src/akili/export/tflite.py` | `TFLiteConverter` (dynamic-range quantisation) and `tf.lite.Interpreter` | `export/onnx.py`: `torch.onnx.export`, then rayo-convert |
| `src/akili/export/trainable.py` | `categoriser-train.tflite` with `train`, `infer`, `save`, `restore` signatures and hand-written gradients for the two dense layers | a written spec and vectors for that SGD step (torch computes the reference); the phone's learning comes with rayo's or Dart's backward pass ([0020](0020-rayo-our-own-inference-engine.md)) |
| `src/akili/export/rayo.py`, `ask/reader_weights.py`, `ask/writer_weights.py` | `convert_tflite`: rayo-convert reads TFLite; weights' `source` is the TFLite file | convert the ONNX file; `source` names it |
| `src/akili/export/manifest.py`, `size.py` | manifest "next to every exported TFLite file"; size budget on TFLite bytes | manifest and budget on the `.weights` file |
| `src/akili/reader/train.py` | message reader: Keras 1-D CNN (three convolutions, one dilated by 4), `fields` and `kind` heads; about 160k parameters, about 200 KB TFLite, 512 KB budget | torch |
| `src/akili/ask/reader.py` | Ask reader: two embeddings, five `Conv1D` (dilations 3 and 2), masked max and mean pooling, four dense layers; 239,589 parameters; 400 KB budget; `ask-reader-probe` reads a `.tflite` | torch |
| `src/akili/ask/writer.py` | Ask writer: Keras causal transformer (width 128, 4 heads, 3 layers, feed-forward 256, position table, GELU, layer norm); 1.5 MB budget; samples with the Keras and TFLite models | torch |
| `src/akili/tracking.py` | MLflow (not Google) | unchanged |
| `src/akili/cli.py`, `config.py` | `.tflite` defaults and help text, `trainable` and `score_exported` options | ONNX and `.weights` |
| tests | `test_tf_training.py`, `test_tf_federated.py`, `test_tf_trainable.py`, `test_tf_audit.py`, the `tf`-marked tests in `test_ask.py` and `test_weights.py`; `test_reader.py`, `test_size_and_cli.py` and `test_vectors.py` mention TFLite | ported to torch, or removed with what they test |
| `tool/check`, `.github/dependabot.yml` | full check installs TensorFlow; dependabot `ml` group lists `tensorflow*`, `keras*`, `dp-accounting` | torch, onnx, opacus |
| docs | `README.md`, `docs/WEIGHTS.md` (the TFLite file "stays the reference"), `docs/ASK.md`, `docs/READER.md`, `docs/FEATURES.md`, `docs/DP-SGD-REVIEW.md`, `docs/PRIVACY-AUDIT.md`, `docs/RELEASE-CHECKLIST.md` | rewritten as each model moves |

Outside pesaclarity-ml:

- **pesaclarity-app**: Ask weights and manifests (`assets/models/ask/`); fixtures recorded with
  TFLite by `tool/ask_reader_decodes.py` and `tool/ask_writer_decodes.py`
  (`test/fixtures/ask/ask_reader_decodes.json`, `ask_writer_decodes.json`); the README's
  provenance; comments naming TFLite in `DartAskReader`, `plain_weights.dart` and the tests;
  the categoriser's `TfliteRuntime` and `categoriser-train.tflite` names. No TensorFlow
  dependency.
- **pesaclarity-backend**: the Flower aggregator (`ngome.fl.server`, FedAvg under SecAgg+ with
  client-side clipping) works on NumPy arrays and has no TensorFlow. Unchanged, as long as the
  order and layout of the arrays stay the same.
- **rayo**: rayo-convert reads TFLite and float32 ONNX (opset 13 or later), and Hugging Face
  language models from safetensors ([CONVERTING.md](https://github.com/pesaclarity/rayo/blob/main/docs/CONVERTING.md)).
  ONNX's 1-D `Conv`, with dilation, `Gemm`/`MatMul`, `LayerNormalization`, `Gelu` and the erf
  pattern, `Gather`, reductions, comparisons and `Where` are mapped, and PyTorch's attention
  export is recognised for the key/value-cache rewrite. Quantised ONNX ops (`QLinear*`,
  `MatMulInteger`) are refused, so **int8 from ONNX does not exist yet**: today's int8 weights
  are TFLite's, read as they are.

## Decision (proposed)

**Move every pesaclarity-ml model, its training and its privacy stack to PyTorch, one model
per pull request, export through ONNX into rayo-convert, and remove TensorFlow, Keras, TFLite
and `dp-accounting` once the last model has moved.**

- **Models and training**: PyTorch on CPU. These models are small and train on a CPU today.
  Each torch module keeps the Keras layer names (`conv_1`, `slot_hidden`, `attention_0`,
  `ln_0_a`), so weight names, the `layers` list the Dart reader checks and the PCWT format
  (version 1) do not change.
- **Export**: `torch.onnx.export` at a pinned opset (17; the converter needs 13 or later), then
  rayo-convert, as for TFLite today. ONNX is the Linux Foundation's (LF AI & Data,
  Apache-2.0). The safetensors path stays for Hugging Face language models (Ask LM); the small
  models are not Llama-shaped.
- **int8 in rayo-convert, from float**: a new option (`--weights int8` for ONNX graphs) that
  quantises in the converter, so the scheme lives in one place:
  - weights symmetric int8 (zero point 0) with the scale axis each tensor has in today's files
    (recorded in their headers), largest magnitude to ±127;
  - activations quantised on the fly as rayo's hybrid kernels do today (`dense` per input row,
    asymmetric; `conv1d` per tensor and per dilation phase, `tensor_phases`);
  - embedding tables as today: int8 with one scale for the table, each row dequantised as it is read.
  Then the Dart reader still runs the files unchanged. The converter's NumPy reference
  (`rayo_convert.ref`) gets the same int8 meaning, and rayo is checked against it bit for bit.
- **DP-SGD**: per-person gradients with `torch.func` (Opacus clips per example, and the
  privacy unit here is one person), with the same Poisson sampling, clipping across all layers,
  Gaussian noise with standard deviation σ × C, and division by the expected batch. Noise comes
  from OS entropy unless `dp.noise_seed` is set, as now. The review in `docs/DP-SGD-REVIEW.md`
  is redone on the torch code: the same measured checks (per-example clipping to about 1e-6;
  noise standard deviation; Poisson batches; counted steps).
- **Accountant: Opacus** (Meta, Apache-2.0) in place of Google's `dp-accounting`:
  `RDPAccountant` with the repository's RDP orders for the figure the gate checks, and
  `PRVAccountant` for the tighter figure now reported as `epsilon_pld`.
- **Federated**: Flower stays (it does not depend on the framework) with a torch trainer; the
  client still sends NumPy arrays in the order and layout the server holds today, so torch's
  `[out, in]` kernels are transposed back to Keras' `[in, out]` at the boundary. The backend
  changes only if that layout is deliberately changed, together, in both repositories.
- **Audits** score the file that ships (the rayo `.weights`), with shadow models exported the
  same way.

### Re-deriving and re-signing the privacy numbers

The limits in `src/akili/resources/tracks.yaml` (app: ε ≤ 8, δ ≤ 1e-6; commercial: ε ≤ 3,
δ ≤ 1e-7; the audit limits) were signed off on 2026-11-05 in pesaclarity-ml#3. The gate
checks the RDP ε; every report also gives `epsilon_pld`. Example from `docs/DP-SGD-REVIEW.md`
for `configs/dp-sgd.yaml` (n = 9,600, q = 0.0133, 375 steps, σ = 1.1, δ = 1e-6): RDP with the
repository's orders 1.830, the library's default orders 1.814, PLD 1.522.

1. Before removing `dp-accounting`, write its RDP and PLD ε for every config, a grid of q, σ,
   steps and δ, and each ledger entry, to a fixture.
2. Compute the same with Opacus (RDP with the same orders; PRV). Differences come from the RDP
   to (ε, δ) conversion and the PRV against PLD discretisation; record each.
3. Gate on the Opacus RDP figure. Ledger entries already keep their sampling rate, noise
   multiplier and steps, so totals are recomputed rather than copied.
4. The owner re-signs in #3 (or a new issue): the thresholds do not change, but the code that
   computes the figure they are compared with does. Until then the torch pipeline cannot pass
   the gate.

### Equivalence and gates per model

Retrained torch models will not be bit-identical to the Keras ones. Each must match or beat its
current scores on the same synthetic sets and seeds, and stay within its size budget, measured
on the converted int8 file under rayo:

| Model | Current figures to match or beat | Budget |
| --- | --- | --- |
| Message reader (not in the app yet) | unseen-layout mean F1: amount about 0.79, payee 0.70, date 0.98 (`reader-report.json`); `reader_vectors.v1.json` inputs exact | 512,000 bytes |
| Ask reader v3 | intent / all slots exact: conformance (68) 1.000 / 0.971; misses (62) 0.984 / 0.968; hard held-out (2,000) intent 0.964, payee/account exact 0.926, unseen-word names 0.885; app questions (65) intent 1.000, names 0.959; unseen v1 templates intent 0.868, slot F1 0.848 | 400,000 bytes |
| Ask writer | checker pass rate at temperature 0.8 on 2,000 samples about 99.4% (98.4% for `spent`); varied replies about seven in ten; `ask_examples.v1.json` fallbacks, checker and detokenise cases | 1,500,000 bytes |
| Categoriser | `val.accuracy` on the same configs and seeds: the docs publish no figure, so it is recorded from the Keras pipeline before the move; `feature_vectors.v1.json` exact | 512,000 bytes |
| Categoriser with DP | the gate passes; audits within the track limits (the DP-SGD audit run reports ε 2.67 today, `docs/PRIVACY-AUDIT.md`) | as above |

- **Parity of the conversion**: the torch float model and rayo's int8 file give the same
  arg-max labels on the recorded sets, with probabilities within a tolerance set per model
  (*proposed*: 1e-2, the gap between plain floats and int8 that 0019's spike measured).
- **Fixtures re-recorded with no TFLite**: the app's decodes are recorded from rayo running the
  new file, with the torch model as a cross-check (same labels and tokens). The app's rayo must
  match them bit for bit and the Dart reader within 1e-4, as now.
- **Determinism**: fixed seeds, `torch.use_deterministic_algorithms(True)`, a fixed thread
  count; the same code, torch version and machine give the same SHA-256 for the exported
  file, as the Keras export does today. The report records the torch, ONNX and rayo-convert
  versions.

## Stages

One pull request per model, smallest risk first. TensorFlow stays an optional extra until
the last model has moved.

| # | Pull requests | Effort (*estimate*) | Risk |
| --- | --- | --- | --- |
| 1 | rayo: int8 from float ONNX, its reference and tests. ml: the `[torch]` extra, `export/onnx.py`, the parity harness | 3 to 5 days | medium: the int8 scheme must match rayo's kernels and the Dart reader |
| 2 | Message reader. Not in the app, so nothing there changes | 2 to 3 days | low |
| 3 | Ask reader. App: new weights and manifest, re-recorded `ask_reader_decodes.json`, `tool/ask_reader_decodes.py` records from rayo and torch, README provenance | 3 to 5 days | medium: shipped, fallback to the Dart reader and the rules |
| 4 | Ask writer. App: new weights, re-recorded `ask_writer_decodes.json`, `tool/ask_writer_decodes.py`, README | 3 to 5 days | medium: the attention rewrite must still fire on torch's export |
| 5 | Categoriser with the privacy stack together: torch trainer, DP-SGD, Opacus accounting, ledger, audits, federated simulation, the dense-layer SGD spec in place of the trainable TFLite. Re-signing in #3 | 6 to 10 days | high: privacy review; the app does not run the categoriser yet |
| 6 | Remove TensorFlow, Keras, TFLite and `dp-accounting`: extras, lock file, mypy overrides, pytest marker, dependabot, `tool/check`, the `test_tf_*` files, docs. rayo's converter tests keep their recorded TFLite files (rayo reads TFLite for others) | 1 to 2 days | low |

Releases go through `dev` and release-please as now; each model's weights change version. In
the app each change is one pull request: weights, manifest, fixtures, README, the two
recording tools; comments that name TFLite as the reference are reworded (the int8 arithmetic
stays rayo's).

## Consequences

- One framework for every model, Ask LM included; no TensorFlow, Keras, TFLite or Google
  library in pesaclarity-ml. The full check gets lighter (*estimate*).
- rayo-convert becomes the one place that quantises the small models.
- Every shipped model is retrained once; scores move a little either way, and gates hold them.
- The DP-SGD review and the privacy sign-off are redone.

## Risks and open questions

- **int8 parity** between the converter's quantisation and the Dart reader's arithmetic:
  tested on the recorded questions before the Ask reader moves.
- **Torch's ONNX export** may emit patterns the converter does not yet map (masked pooling,
  the attention path, the position table); each is found in stage 1 or the model's pull
  request and added to the converter as a general mapping.
- **Opacus's accountant** may give slightly different ε than `dp-accounting`; the gate needs
  the owner's sign-off on it.

Questions for the owner:

1. **Opacus** (Meta, Apache-2.0) as the accountant, and re-signing in #3?
2. **Old TFLite files**: keep the last TFLite releases as archived artefacts for comparison,
   or drop them?
3. **The writer**: move it in stage 4, or leave it on Keras until Ask LM replaces it? It
   stays Ask's fallback after 0022, so this plan moves it.
4. **Order**: the message reader first, as proposed, or the Ask reader first, because it
   ships?

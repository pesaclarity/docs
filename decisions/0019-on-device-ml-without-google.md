# 0019. On-device ML without Google: one runtime for Play and F-Droid

Date: 2026-11-17 · Status: Proposed

## Context

We want the F-Droid build to have the same on-phone models as the Play build, the app to work on Huawei and other phones without Google services, and the option of dropping Google from the Play build too.

Today every model runs through `tflite_flutter` 0.12.1 (Apache-2.0). It does not need Google Play services: it bundles the LiteRT (TFLite) runtime. But on Android its `build.gradle` pulls two prebuilt AARs from Google's Maven, `com.google.ai.edge.litert:litert:1.4.0` and `litert-gpu:1.4.0` (each also depends on its `-api` AAR). They carry `libtensorflowlite_jni.so` (about 4.3 MB on arm64) and `libtensorflowlite_gpu_jni.so`; the plugin's Dart code opens `libtensorflowlite_jni.so` through FFI, and the app already drops the GPU library from the APK. Its Android CMake build is commented out, so nothing is compiled from source.

F-Droid builds apps from source and does not accept prebuilt native libraries (0006, where we removed PDFium for the same reason). `tool/prepare_foss.sh` removes PDFium but not `tflite_flutter`, so the `foss` build still carries the prebuilt LiteRT library. The app can already run without it (a null runtime makes Ask and the categoriser use their rules), but F-Droid users would then get no models. What we know, and what we must confirm with F-Droid before relying on it:

- Known: a prebuilt `.so` inside a Maven AAR is the same problem as PDFium's download; LiteRT itself is open source (Apache-2.0) and can be built with Bazel.
- To confirm: whether F-Droid's build server can run a full LiteRT Bazel build within its time and disk limits, and whether a runtime we build in the recipe can be reproducible enough for F-Droid to ship our signed APK.

The models today, all small, all exported as TFLite by pesaclarity-ml:

| Model | Ships | Size | Layers (TFLite ops) |
| --- | --- | --- | --- |
| Ask reader | in the app | 272 KB, 240k parameters | two embedding lookups, five 1-D convolutions (two dilated, run as `SPACE_TO_BATCH_ND`), four dense layers, ReLU, residual adds, masked max and mean pooling, softmax. int8 weights with float inputs (dynamic-range quantised) |
| Ask writer | in the app | 653 KB, 532k parameters | a small transformer: embedding, 13 dense layers, attention (`BATCH_MATMUL`, softmax), layer norm (`MEAN`, `SQUARED_DIFFERENCE`, `RSQRT`), GELU |
| Message reader | not yet (pending the model licence) | 199 KB, 180k parameters | as the Ask reader, with three convolutions |
| Categoriser | not yet (after the privacy gate) | about 82 KB, 68k parameters (smoke export) | embedding lookups, sum and mean pooling, two dense layers, softmax. Its trainable copy adds `train`, `save` and `restore` signatures for learning on the phone |

## Options

- **(a) Keep TFLite, build it from source in the F-Droid recipe** with Bazel. Same runtime as Play, no model changes. But it is a large C++ build with its own toolchain, slow and heavy on F-Droid's servers, and hard to make reproducible. Huawei phones are fine either way.
- **(b) A small pure-Dart inference package** for the ops our small models use. No native code at all, so F-Droid builds it like any Dart code, and Play, F-Droid and iPhone share one path. Slower than TFLite's optimised kernels, and we own every op we add.
- **(c) An open-source native runtime built from source with CMake**, for larger models later: llama.cpp/ggml (MIT), ONNX Runtime Mobile (MIT), ExecuTorch (BSD), ncnn (BSD) or MNN (Apache-2.0). All build with CMake and the NDK, a much smaller build than LiteRT's Bazel one. Still to check per runtime: build time on F-Droid's servers, size, and the ops our models need.
- **(d) Write a full general runtime ourselves.** Rejected: years of work to match these, and slower than any of them.

## Spike: the Ask reader in pure Dart

pesaclarity/pesaclarity-app, branch `claude/sweet-albattani-bs9rdz` (a spike, not for release):

- A script reads the weights out of the shipped `ask-reader.tflite` into a plain file (int8 tensors with their scales, 248 KB) and a layer list.
- About 400 lines of Dart run the forward pass. Run the way TFLite runs this model (each layer's input quantised to int8 on the fly), it gives the same labels as TFLite on all 30 recorded questions, intent and every token's slot, with probabilities at most 3e-5 apart (the recording rounds to 1e-5). Plain floats with the same weights also give the same labels, at most 1.2e-2 apart.
- On one core of a 2.1 GHz Xeon (AOT, `dart compile exe`): weights load in 3 to 5 ms; one question takes 14 ms with plain Dart doubles and 5 to 6 ms with `Float32x4` SIMD. TFLite on the same machine takes 0.06 ms (XNNPACK) to 0.25 ms (built-in kernels): pure Dart is 20 to 80 times slower, but fast enough for one question at a time.
- No phone was measured. Estimate only: a low-end phone core is perhaps 3 to 6 times slower than this core, so roughly 15 to 90 ms per question.

## Decision (proposed)

One Google-free ML layer for both builds, behind the app's existing runtime interfaces:

- **Pure Dart for today's small models**: the Ask reader, the message reader, the categoriser and, if it is fast enough, the Ask writer. No native ML library in either build.
- **An open-source native runtime, built from source**, only when a model is too big or slow for Dart. We pick one of (c) then, after measuring it.
- **pesaclarity-ml keeps exporting TFLite** (its tests and evaluation use it) and adds a plain-weights export with a layer list and conformance outputs, as the spike's script does, so the app can check every release against recorded outputs.
- **Large models are downloaded after install**, not bundled, and checked against the manifest's size and SHA-256 before use, as the categoriser already is. Small models stay in the APK.

## Consequences

- The F-Droid build gets the same Ask, reader and categoriser as Play, with no recipe changes for ML. Huawei and other no-Google phones are covered by both builds.
- `tflite_flutter` and the LiteRT AARs leave the app once each model has moved over, which also saves about 4 MB of native library per chip.
- We own the Dart ops and their tests: every new layer type needs a Dart version checked against TFLite's outputs. The categoriser's learning on the phone needs a Dart backward pass for its two dense layers, which TFLite now gives us for free.
- To match TFLite closely the Dart code copies its int8 handling; models exported in plain float32 would be simpler to match.

## Risks and open questions

- **Low-end phones.** Measure the reader and the writer on a 2 GB, 32-bit phone before moving anything. The writer decodes word by word, so it may be too slow in Dart.
- **F-Droid.** Confirm with F-Droid that pure-Dart inference needs nothing special, and, for (c), what build time and disk their servers allow and whether the native build can be reproducible.
- **Which native runtime**, when we need one: decided by measured build time, size, speed and op coverage, in its own record.
- **Downloads.** Where large models are hosted, and how the F-Droid build fetches them without a tracker or a Google service.

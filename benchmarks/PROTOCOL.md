# Benchmark protocol

Reproducible measurement is part of the product story. Every published number
follows this protocol; any deviation is disclosed alongside the number.

## Setup

1. Lock input resolution and camera (record device model).
2. Fix ambient conditions per scenario (see matrix below).
3. Warm the runtime: discard the first 200 frames.
4. Measure **≥ 1,000 consecutive frames**.
5. Report **median and p95** end-to-end latency (camera photon → virtual-cam frame),
   plus FPS, peak RSS, and model package size.
6. Publish with every result: device, model checkpoint hash, runtime
   (ONNX Runtime / HailoRT), quantization settings, driver versions.

## Scenario matrix

| # | Scenario | Why |
|---|---|---|
| 1 | Normal indoor light | Baseline |
| 2 | Low light | Noise robustness |
| 3 | Backlight (window behind) | Mask failure mode |
| 4 | Fast head motion | Temporal smoothing stress |
| 5 | Hair detail / loose hair | Edge refinement quality |
| 6 | Glasses | Transparency edge case |

## Acceptance targets (Stage II, UGen300 hardware)

- End-to-end latency p95 < 50 ms @ 720p
- Sustained ≥ 25 FPS with all three effects on
- Model package < 10 MB
- No frame upload at any point (verify with a local packet capture during the run)

Targets are not results until measured under this protocol.

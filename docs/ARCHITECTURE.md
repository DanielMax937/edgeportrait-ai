# Architecture

## Target hardware

ASUS UGen300 (per ASUS CES 2026 announcement): USB-C edge AI accelerator,
Hailo-10H, 40 TOPS INT8-class compute, 8 GB dedicated LPDDR4, Windows / Linux /
Android support, ONNX / TensorFlow / PyTorch workflows.

## Pipeline stages

1. **capture** — fixed-resolution frame buffers (720p preview in MVP); isolates
   camera I/O jitter from inference.
2. **segment** — person segmentation model, full INT8. Output: portrait mask +
   confidence. Temporal smoothing across frames suppresses flicker; edge
   refinement protects hair detail and glasses.
3. **relight** — face/foreground light map; adjusts exposure, warmth, and
   key-light direction from the mask and a lightweight face-brightness estimate.
4. **stylize** — one preset at a time (MVP): color/tone/texture LUT-class transform
   applied inside the foreground mask. Identity-preserving by construction.
5. **compositor** — background replace/blur + foreground composite + metrics
   overlay (latency, FPS) for the benchmark story.
6. **virtualcam** — virtual-camera sink consumed by OBS / Meet / Zoom / etc.

Stages communicate over bounded async queues; zero-copy buffer hand-off where
the platform allows, so USB/host compositing does not become the bottleneck.

## Model choices (hardware-aware)

- Segmentation: MobileNetV2-class backbone or equivalent, INT8 quantized.
  Rationale: fits the <10 MB package target and the INT8 datapath.
- Relight/stylize: closed-form / LUT-class ops on CPU — no second neural net in
  the MVP hot path.
- All models consumed as ONNX for runtime portability (CPU baseline in Stage I,
  Hailo path in Stage II).

## Latency budget (targets, to be measured in Stage II)

End-to-end target: **< 50 ms** camera-to-output at 720p. Model package target:
**< 10 MB**. These are proposal targets; Stage II measures them on real UGen300
hardware per `benchmarks/PROTOCOL.md`.

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| Unsupported ONNX operators on the accelerator | Operator audit before porting; calibration set ready |
| Poor INT8 accuracy (esp. hair edges) | Calibration set, confidence masks, temporal smoothing |
| USB / host compositing bottleneck | Async queues, zero-copy, fixed-resolution buffers |
| Solo-builder scope creep | MVP frozen after one style preset; measurement > breadth |

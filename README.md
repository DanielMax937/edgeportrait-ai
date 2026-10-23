# EdgePortrait AI

**Private, real-time portrait effects on the edge.**
Turn any ordinary webcam into a creator-grade virtual camera — without uploading a single frame.

> Submission for the **2026 ASUS UGen AI League Innovation Hackathon**
> Track: **Battlefield Lightning** · Theme: **Creator AI** · Participation: **Global Online, solo**
> Participant: Daniel Cao

EdgePortrait AI is a local virtual-camera engine for streamers, remote professionals,
educators, and solo creators. It combines **person segmentation**, **intelligent
relighting**, and **live visual stylization** in one low-latency pipeline targeted at
the **ASUS UGen300** edge AI accelerator (Hailo-10H, 40 TOPS, 8 GB LPDDR4, USB-C,
ONNX/TensorFlow/PyTorch workflows).

**Design principles**
1. The effect must never block the conversation — latency budget is king.
2. Raw video frames stay local by default. No account, no face template, no cloud.
3. One click switches looks — no pipeline rebuilds.

> **Honesty note:** this repository currently holds the Stage I proposal scaffolding.
> All latency / FPS / model-size figures in the proposal are **targets for Stage II
> measurement on real UGen300 hardware**, not measured results. The Stage I demo
> shows the software interaction on a CPU/ONNX baseline and is labeled as such.

## Pipeline

```
webcam ──▶ capture ──▶ person segmentation (INT8) ──▶ relight ──▶ stylize ──▶ composite ──▶ virtual camera ──▶ OBS / Meet / Zoom
                              │                          ▲              ▲
                              └──── temporal smoothing ──┘              │
                                                     metrics overlay ──┘
```

- **Segmentation** — portrait-aware mask with temporal stabilization and edge refinement.
- **Relighting** — face/foreground light map adjusts exposure, warmth, key-light direction.
- **Stylization** — one lightweight preset changes color/tone/texture without replacing identity.
- **Fallback** — if accelerator inference is unavailable, effects degrade gracefully;
  the camera never freezes.

## MVP scope (Stage II)

Included: single-person portrait · indoor lighting · 720p preview · background
replace/blur · face-aware relighting · one live style preset · preset switching ·
metrics overlay · virtual-camera output.

Deliberately excluded: multi-person tracking, model training UI, preset marketplace,
mobile app, cloud sync.

## Repository layout

```
edgeportrait-ai/
├── README.md                  # this file
├── LICENSE                    # MIT
├── requirements.txt           # Python dependencies (Stage II)
├── docs/
│   └── ARCHITECTURE.md        # pipeline design, model choices, risk mitigations
├── benchmarks/
│   └── PROTOCOL.md            # reproducible latency/FPS/memory benchmark protocol
└── src/
    └── edgeportrait/
        ├── __init__.py
        ├── capture.py         # webcam capture, fixed-resolution buffers
        ├── segment.py         # person segmentation (INT8 ONNX)
        ├── relight.py         # face-aware relighting
        ├── stylize.py         # live style preset
        ├── pipeline.py        # async stage queues, zero-copy where available
        ├── compositor.py      # effect compositing + metrics overlay
        └── virtualcam.py      # virtual-camera output sink
```

## Benchmark protocol (summary)

Lock input resolution and camera → warm the runtime → measure ≥1,000 consecutive
frames → report median and p95 latency → cover normal / low / backlight, fast
motion, hair detail, glasses → publish device, model, runtime, and quantization
settings with every result. Full protocol: [`benchmarks/PROTOCOL.md`](benchmarks/PROTOCOL.md).

## Roadmap

- **Stage I** (proposal): this repo + ≤20-page deck + ≤3-min demo video (CPU/ONNX baseline).
- **Stage II** (build, on UGen300 hardware): operator audit → INT8 calibration →
  pipeline integration → benchmark on device → weekly evidence drops.
- **Post-hackathon**: preset marketplace, SDK integration, mobile support,
  creator-app plug-ins.

## Privacy

No face recognition — the pipeline segments and enhances the current frame; it
does not identify anyone. Effects are opt-in, visible in preview, reversible.
The MVP is tuned for one foreground person and states unsupported conditions.

## References

- ASUS — "ASUS Unveils Full-Spectrum AI and Always Incredible Innovations at CES 2026"
  (UGen300 specifications): https://www.asus.com/us/news/otappc57ilrhehpy/
- 2026 ASUS UGen AI League Innovation Hackathon (official event page):
  https://contest.bhuntr.com/en/39jg9vimiynrhlksze/

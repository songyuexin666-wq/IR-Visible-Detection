# IR-Visible Detection

<p align="center">
  <b>Query-Level Evidence Selection for Infrared-Visible Multimodal Object Detection</b>
</p>

This repository documents an ongoing research project on **infrared-visible multimodal object detection**. The project studies how an object detector can decide, at the **query level**, whether a prediction should rely more on RGB evidence, infrared evidence, or their joint representation.

The current implementation is built around **RT-DETR-style query-based detection** and evaluates multimodal detection on datasets including **M3FD** and **FLIR**.

---

## Motivation

Infrared and visible images provide complementary information:

- **RGB** captures texture, color, and fine spatial details under favorable illumination;
- **Infrared** remains informative under darkness, glare, and illumination changes.

However, naive feature fusion assumes that both modalities are equally useful for every object and every scene.

In practice:

```text
different object
      +
different scene
      +
different query
      ↓
different evidence requirement
```

This project therefore focuses on **query-adaptive evidence selection** rather than static feature fusion.

---

## Core Idea

For each decoder query, the model evaluates multiple evidence sources:

```text
RGB evidence
     │
IR evidence
     │
Joint RGB-IR evidence
     │
     ▼
Query-level evidence assessment
     │
     ▼
Adaptive routing / fusion
     │
     ▼
Detection prediction
```

The goal is not simply to answer:

> Which modality is globally better?

but instead:

> Which evidence source is most useful for this specific query?

---

## Evidence Action Bank

The research is organized around a small action bank for multimodal evidence selection.

A query may choose among:

```text
RGB
IR
Joint RGB + IR
KEEP / no modification
```

The decision is conditioned on whether a change in evidence is expected to improve the current prediction.

This leads to a progression from:

```text
static fusion
    ↓
learn whether a query is correctable
    ↓
select positive evidence action
    ↓
retain only high-confidence positive actions
```

---

## Research Progression

The current ablation path can be summarized as:

### ab1 — KEEP baseline

No query-level evidence intervention.

### ab2 — Correctability modeling

Introduce an action bank that learns whether a query is potentially correctable by switching evidence.

### ab3 — Positive-gain routing

Only retain evidence actions that produce a positive improvement.

### ab4 — High-confidence positive routing

Require the evidence action to be both beneficial and sufficiently reliable before applying it.

This design attempts to reduce harmful modality switching and unstable fusion behavior.

---

## Architecture

The current detector uses a ResNet + RT-DETR style pipeline with multimodal feature extraction and query decoding.

Conceptually:

```text
RGB image ─────► RGB encoder ────┐
                                 │
                                 ├──► Multimodal feature interaction
                                 │
IR image ──────► IR encoder ─────┘
                                           │
                                           ▼
                                     DETR decoder
                                           │
                                           ▼
                                      Query features
                                           │
                                           ▼
                                Evidence Action Bank
                                  RGB / IR / Joint
                                           │
                                           ▼
                                     Final prediction
```

---

## Frequency-Aware Branch

The project also explores a frequency-oriented multimodal branch.

Observed feature statistics show that the branch changes the representation distribution rather than simply duplicating the original features.

Representative diagnostics include:

```text
low-frequency norm:   16.80 → 18.34
high-frequency norm:  15.57 → 17.53
feature similarity:    0.164 → 0.135
```

The observed fusion weight remains close to balanced:

```text
low_weight ≈ 0.50
```

These diagnostics suggest that the frequency branch is active, although its direct detection gain remains limited and continues to be investigated.

---

## Current Challenges

The main remaining issue is **localization quality**, especially for small objects.

The project has observed that:

- AP@0.5 can be competitive,
- AP@0.75 remains comparatively weaker,
- frequency fusion alone does not fully solve boundary precision,
- evidence selection needs to improve both classification confidence and spatial localization.

This motivates further work on:

- edge-aware localization,
- boundary-sensitive supervision,
- query-level localization confidence,
- PiDiNet / Gated-SCNN-style edge priors,
- stronger cross-modal geometric consistency.

---

## Datasets

### M3FD

M3FD is used as a primary benchmark for infrared-visible multimodal detection.

A representative reference configuration reaches approximately:

```text
AP@0.5 ≈ 87.0
```

for the WD-FQDet-style baseline used in project comparisons.

### FLIR

FLIR is used to evaluate multimodal detection in thermal-visible driving scenes.

Representative baseline performance is approximately:

```text
AP@0.5 ≈ 50.1–50.2
```

Exact values depend on preprocessing, backbone, split, and evaluation protocol.

---

## Research Questions

This project studies three core questions:

### 1. When should a query trust RGB?

RGB may be more informative when texture and local structure are strong.

### 2. When should a query trust infrared?

Infrared may be more reliable under low illumination or weak visible contrast.

### 3. When is joint evidence actually helpful?

Joint fusion should be applied only when it improves the current query rather than by default.

---

## Planned Evaluation

The final evaluation will compare:

- RGB-only detector;
- IR-only detector;
- static RGB-IR fusion;
- frequency-aware multimodal fusion;
- query-level action routing;
- positive-gain routing;
- high-confidence positive routing.

Metrics include:

- AP@0.5
- AP@0.75
- AP@0.5:0.95
- small-object AP
- per-class AP
- routing gain
- harmful-action rate
- evidence-selection confidence

---

## Planned Repository Structure

```text
IR-Visible-Detection/
├── datasets/
├── models/
│   ├── rgb_encoder/
│   ├── ir_encoder/
│   ├── fusion/
│   └── query_router/
├── configs/
├── scripts/
├── evaluation/
├── assets/
└── README.md
```

---

## Status

**Research in progress.**

The current stage focuses on improving the transition from multimodal feature fusion to **reliable query-level evidence selection**, with particular emphasis on localization accuracy and avoiding harmful modality switching.

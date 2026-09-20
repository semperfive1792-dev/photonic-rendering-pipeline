# Photonic Rendering Pipeline

**What if a display didn't need HDMI, DisplayPort, a scaler, a T-CON, or a pixel matrix?**

This repository proposes a display architecture where the path from render output to visible light is optical, not digital. No framebuffer, no display cable, no digital-to-analog conversion at the panel. The idea is a question, not a finished design — but a question that may be worth asking.

---

## The Problem

Today's display chain has 11 stages between the GPU and your eyes:

> GPU render → framebuffer → display encoder → cable (HDMI/DP) → receiver → scaler → T-CON → column drivers → DAC → pixel matrix → light

Each stage adds latency, power, and cost. Each stage exists because it solved a real problem — but nobody has asked whether the whole chain can be replaced, because the alternative wasn't possible until recently.

## The Proposal

Replace the digital display chain with an optical one:

> Photonic computation → optical transport → laser excitation → light

Six stages instead of eleven. The transport layer is light, not copper. The modulation is optical, not electrical. The display surface emits light directly from laser excitation — no pixel matrix, no column drivers, no DAC.

This is **not** a fully photonic GPU. It is a heterogeneous system: electronic for logic and memory, photonic for transport and modulation.

## What's Real

This proposal builds on seven existing technologies:

| Technology | Source | What it does |
|------------|--------|-------------|
| ACCEL | Tsinghua, *Nature* 2023 | Analog photonic chip — 3000× faster than A100 for classification (not 4K rendering) |
| OPCA | Tsinghua, *Optica* 2024 | Photonic chip integrating sensing and computing |
| SEECHIP | ICPP 2023 | Photonic accelerator compiler |
| MIT ski-jump | *Nature* 2025 | On-chip free-space beam scanning — 68.6 M spots/s·mm² |
| TriLite Trixel 3 | Display Week 2026 (prototype) | RGB laser beam scanning, 214% sRGB |
| Prysm LPD | Commercial (signage) | Laser-phosphor display, >80,000:1 contrast |
| Brilliance Laserchip | 2026 | Laser chips for AR display |

None of these alone is a display. The proposal gives them an integration framework.

## What This Is Not

- **Not a finished architecture.** Pixel addressing is unsolved — 6 G spots/s for 4K@240. Three variants are proposed (beam scanning, emitter array, tiled scanners), none selected.
- **Not a claim that the industry is stupid.** Every layer in the current chain solves a real problem. This proposal asks whether some of those problems have a simpler solution in the optical domain.
- **Not "zero latency."** End-to-end latency is dominated by the render itself (~8 ms at 120 Hz). The target is **display-chain latency** — from render output to photon — not scene-to-photon.
- **Not "infinite contrast."** OLED achieves true black by turning pixels off. The advantage here is a combination of brightness, contrast, and gamut, not any single metric.

## Simulation

`simulation/latency_model.py` — an architectural model comparing conventional and photonic display chains.

| Metric | Conventional | Photonic | Reduction |
|--------|-------------|----------|-----------|
| Display-chain latency | ~3.2 ms | ~0.73 ms | 77% |
| Power | 335 W | 78 W | 77% |
| Stages | 11 | 6 | — |

**This is an architectural illustration, not experimental proof.** The numbers are derived from component specifications and engineering estimates, not measured photonic hardware.

```bash
python3 simulation/latency_model.py
```

## What to Read

- **[Photonic_Rendering_Pipeline.md](Photonic_Rendering_Pipeline.md)** — full technical document (45 sections, ~62K chars). Architecture, addressing budget, experimental roadmap, 8 "Where I'm Probably Wrong" items, 7 falsifiable predictions.
- **[VISION.md](VISION.md)** — market context for non-engineers. Why this might matter commercially, and why it's published under CC BY 4.0.
- **`simulation/`** — Python models for latency and power budget.

## The Honest Core

The strongest part of this document is not the proposal — it's the questions it asks honestly:

1. Can 8 million pixels be addressed optically at 240 Hz without recreating a T-CON?
2. Can analog optics deliver 10-bit HDR precision (4–8 bits is typical)?
3. What does photonics add for latency that digital scanout and racing the beam don't?
4. Input traffic is 8–30× larger than the output it replaces. Why does an optical interface win?

If these questions have bad answers, the proposal fails. If they have interesting answers, it's worth building.

## Publications

📄 **Archived version:** [Photonic Rendering Pipeline: A Proposal for an All-Optical Display Chain](https://doi.org/10.5281/zenodo.22862885) (CC-BY 4.0)

## License

CC BY 4.0. The value is in integration, not in patenting components. This is a defensive publication — it exists so that nobody can patent the architecture, including the author.

---

*Not an engineer? Start with [VISION.md](VISION.md).*
*Engineer? Start with the full document, then tell me where I'm wrong.*

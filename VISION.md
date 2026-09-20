# VISION

## Market Context for the Photonic Rendering Pipeline

> This document is a companion to `Photonic_Rendering_Pipeline.md`. The technical
> architecture, references, and open questions live there. This file speaks to
> those who decide whether to fund, build, or adopt — not to the engineers who
> will critique the photonics.

---

## 1. Where the Display Industry Is

The display industry is in a cycle of incremental refinement. Recent product
announcements at CES 2025 and CES 2026 focus on:

- Higher refresh rates (500 Hz OLED at 1440p, announced at CES 2026)
- Brightness increases through tandem OLED stacks (LG Tandem WOLED Gen 4,
  Samsung Penta Tandem QD-OLED)
- New cable standards (HDMI 2.2 at 96 Gbit/s, DisplayPort 2.1b with active
  cables)
- Higher resolution (5K and 6K monitors entering the mainstream)

None of these are architectural changes. Each one adds layers — more drivers,
more bandwidth, more processing — to the same fundamental pipeline: GPU renders
a frame, encodes it, sends it over a cable, a scaler decodes it, a T-CON drives
a pixel matrix.

OLED monitor shipments reached approximately 2.7 million units in 2025 —
about 2.1% of the 133.4 million total monitor market. TrendForce projects 6.9
million by 2028, still only ~5.4%. MicroLED display revenue was $52.4 million
in 2025, projected to double to $105.4 million in 2026 — real growth, but from
a base that is a fraction of a percent of the display market.

The laser phosphor display market (Prysm's technology) was valued at
approximately $10.67 billion in 2026 with a projected CAGR of 30.64% through
2035 — significant, but concentrated in commercial signage and video walls,
not consumer displays.

The point: the industry is healthy but architecturally stagnant. Growth comes
from process improvements and market expansion, not from new display
architectures.

---

## 2. What This Proposal Identifies

A structural discontinuity: a display chain that does not require HDMI,
DisplayPort, a scaler, a T-CON, or a conventional pixel matrix. Not a better
version of what exists — a different chain with fewer intermediate
representations between render and light.

The technical document is explicit about what this does and does not claim:

- **Does not claim** a finished product or a working prototype.
- **Does not claim** that the industry is "doing things stupidly." Each
  conventional layer was added to solve a real problem: interoperability,
  bandwidth, timing, cost.
- **Does not claim** "infinite contrast" or "zero latency" as physical
  certainties. OLED already achieves true black by turning pixels off. The
  advantage of this architecture is in the combination of high brightness
  (laser-driven), wide gamut (narrowband laser), and reduced display-chain
  latency — not in a single spec that beats everything else.
- **Does not claim** that photonic computing replaces GPU rendering. The
  input data (G-buffer, motion vectors, neural features) is 8–30× larger than
  the display output. The E/O conversion does not disappear — it moves from
  the display cable to the chiplet interface.

What it does identify is a path where some functions of the conventional
display chain migrate to the optical domain, and where some layers become
unnecessary. Whether this is technically viable is an open question — the
technical document lists 8 specific areas where the author may be wrong.

---

## 3. Why the Open License Matters

The repository is published under CC BY 4.0. Anyone can use, modify, and
build on the architecture — including commercially.

This is a deliberate choice, not naivety:

- **The value is in integration, not in components.** The individual
  technologies (ACCEL, OPCA, MIT ski-jump, Prysm LPD, TriLite Trixel 3) are
  being developed independently by separate teams. The contribution of this
  proposal is the integration framework — how they connect, what interfaces
  they share, what functions become unnecessary when they work together.
- **Patenting the architecture would prevent the distributed research model.**
  If one entity owned the architecture, no lab would build toward it without
  licensing. CC BY 4.0 removes that barrier.
- **Defensive publication prevents lockout.** Publishing under an open
  license establishes prior art. Others can build on the work; no one can
  patent the architecture and prevent the original author or anyone else from
  using it.
- **First-mover advantage is in execution, not in ownership.** Under CC BY
  4.0, all participants have equal access. The advantage goes to whoever
  validates, prototypes, and ships first — not to whoever holds the patent.

---

## 4. What a Decision-Maker Should Look At

| Question | Where to find the answer |
|----------|------------------------|
| Is the physics real? | Technical doc, Section 26 — existing technologies with measured results |
| What are the biggest risks? | Technical doc, Section 31 — "Where I Am Probably Wrong" (8 items) |
| How would this be tested? | Technical doc, Section 27 — 5-stage experimental roadmap |
| What does it cost? | Technical doc, Section 42 — cost tiers (hardware-only, order-of-magnitude, excludes staff) |
| What can be built today? | Technical doc, Section 42.4 — transitional device (FPGA → fiber → laser-phosphor) |
| What can be tested without hardware? | `simulation/latency_model.py` — architectural illustration of latency and power budget |
| Has anyone built parts of this? | Technical doc, Section 26 — 7 technologies with published results |
| Where does this contradict itself? | Technical doc, Section 31 — self-identified weak points |

---

## 5. What the Market Would See

If the technical validation succeeds, the market would see a display that:

- **Does not use HDMI or DisplayPort.** No licensing fees (HDMI: $10,000/year
  + $0.04–$0.15 per port for manufacturers shipping >10,000 units/year).
  No cable bandwidth ceiling.
- **Does not use a conventional pixel matrix.** No T-CON, no row/column
  drivers, no scaler — but the addressing function does not disappear. It
  migrates to beam scanning (Variant A), an emitter array (Variant B), or a
  tiled scanner array (Variant C). The technical document is explicit that
  this is the central unsolved problem.
- **Generates light directly from laser excitation.** Laser-phosphor (Prysm)
  or direct RGB laser (TriLite). Brightness and gamut come from the light
  source, not from a backlight filtered through an LCD.
- **Reduces display-chain latency** — not end-to-end latency, which is
  dominated by GPU render time. Display-chain latency is the time from frame
  completion to photon emission. The simulation model estimates a reduction
  from ~3.2 ms (conventional display chain) to ~0.73 ms (photonic chain),
  but this is an architectural illustration, not a measurement.

This is not "another monitor with better specs." It is a display built on a
different chain of intermediate representations. Whether that difference
translates into a market advantage depends on engineering validation and
manufacturing cost — neither of which can be assessed from an architectural
concept alone.

---

## 6. Suggested Next Steps for Interested Parties

### For research labs
- Review Section 27 (Experimental Roadmap). Stages 1–2 can be tested at
  university scale ($10–50K, hardware-only).
- The distributed research model (Section 42.2) means no single lab needs to
  build the whole system. Each group holds one piece. The proposal provides
  the integration framework.
- CC BY 4.0 allows using, modifying, and publishing results without
  licensing negotiations.

### For corporate R&D teams
- Review Section 42.4 (Transitional Device). An FPGA → fiber → laser-phosphor
  prototype delivers display-level benefits (brightness, gamut, reduced cable
  latency) without waiting for photonic computing to mature.
- The transitional device is a product before the full architecture is ready.
  It uses existing components (Prysm LPD panels, fiber optic links, FPGA
  controllers) and requires no new physics.

### For the author
- Upload to Zenodo to obtain a DOI. This provides a citeable, timestamped
  identifier stronger than a GitHub commit hash. Zenodo is free, operated by
  CERN, and integrates directly with GitHub releases.
- The technical document should remain the primary artifact. This file
  (VISION.md) is supplementary and should not be cited in technical contexts.

---

## 7. References Used in This Document

| Source | Type | Used for |
|--------|------|----------|
| TrendForce via 4k-monitors.ru | Market analysis | OLED monitor shipments (2.7M in 2025, 2.1% of market) |
| Omdia via awall.com | Market tracker | MicroLED revenue ($52.4M → $105.4M, 2025–2026) |
| Business Research Insights | Market report | Laser phosphor display market ($10.67B in 2026, CAGR 30.64%) |
| Future Market Insights | Market report | OLED display market ($53.3B in 2025) |
| TriLite official website | Primary source | Best Prototype Award, Display Week I-Zone 2026 |
| Prysm / Commercial Integrator | Primary source | LPD 6K specs (contrast >80,000:1, 75% less power) |
| Habr (HDMI 2.2 / DP 2.1b coverage) | Technical journalism | HDMI licensing fees ($10K/year + $0.04–$0.15/port) |
| Zenodo / CERN | Primary source | DOI assignment, GitHub integration, FAIR compliance |

All technical claims (HDMI 2.1 bandwidth, TriLite status, ACCEL/OPCA scope,
addressing budget, G-buffer sizing, precision limitations) are consistent with
`Photonic_Rendering_Pipeline.md` and its 7 primary-source references.

---

## License

CC BY 4.0 — same as the main repository. Attribution required, commercial
use permitted, no additional restrictions.

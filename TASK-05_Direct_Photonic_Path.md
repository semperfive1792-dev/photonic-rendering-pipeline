# TASK-05: Direct Photonic Rendering-to-Display Path

**Stage:** 5 (Final)
**PRP Reference:** §28 Stage 5, §33.1, §31.3, §46
**Status:** Open — desktop research + architecture specification
**Estimated Effort:** 14–21 days (desktop research); optional lab validation $20K–$100K, 6–12 months
**Prerequisites:** TASK-01 (E/O power budget), TASK-02 (spatial addressing), TASK-03 (optical PSF), TASK-04 (neural rendering chiplet)

---

## §1. Research Question

**Can a complete end-to-end optical path be constructed from electronic control through photonic neural rendering, optical image field, and optical interconnect to an optical display — with no intermediate digital framebuffer?**

This is the integration task. Stages 1–4 validated individual components; Stage 5 asks whether they compose into a single continuous optical pipeline.

**Falsifiable test:** If any segment of the chain requires a digital framebuffer (RAM-based pixel store) between the photonic renderer output and the display surface, the core architectural claim of PRP §28 Stage 5 is falsified. The pipeline may still be partially optical, but the "no intermediate digital framebuffer" claim fails.

**Partial success criteria:** A pipeline with a single O/E/O conversion at the display boundary (but no digital framebuffer) is a partial validation — the display is optical, but the final drive is electronic. This is architecturally weaker than full optical continuity but stronger than the current fully-electronic pipeline.

---

## §2. Scope

This task covers the integration architecture only. It does not re-derive component-level metrics — those are in TASK-01 through TASK-04. Here we address:

1. **Chain composition** — how the five segments connect
2. **Interface losses** — coupling loss at each boundary
3. **End-to-end latency budget** — does the full chain meet display-relevant latency?
4. **End-to-end power budget** — does the full chain meet display-relevant power?
5. **Framebuffer elimination** — at which points can digital storage be removed, and where is it unavoidable?
6. **Synchronization** — how is timing maintained without a framebuffer as clock domain buffer?
7. **Failure modes** — what happens when one segment fails?

---

## §3. The Five-Segment Chain

### 3.1 Segment A: Electronic Control

**Function:** Scene data input, neural network weight loading, laser pump control, system synchronization.

**Input:** Scene description (polygon data, point cloud, or pre-computed light field) in electronic form.

**Output:** Electronic signals driving the photonic neural rendering chiplet (modulator voltages, laser pump currents).

**Key constraint:** This is the only segment where digital electronics are mandatory. The question is not whether electronic control exists — it does — but whether it requires a framebuffer-sized digital store, or only streaming control signals.

**Evidence:** Ayar Labs TeraPHY accepts 24 channels of AIB (960 Gbps total) at ~3 ns latency, with no FEC required [S1]. Intel OCI chiplet achieves 4 Tbps bidirectional at ~5 pJ/bit, co-packaged with CPU [S2]. These are data-plane interfaces; the control plane bandwidth is orders of magnitude lower.

**Framebuffer question:** Electronic control needs a weight store (for the neural network) and a scene data stream. The weight store is static (loaded once, updated rarely). The scene data stream is a flow, not a framebuffer — it arrives per-frame and is consumed immediately. Neither requires a full pixel framebuffer if the rendering is truly streaming.

### 3.2 Segment B: Photonic Neural Rendering

**Function:** Transform electronic scene representation into an optical image field.

**Input:** Electronic control signals (modulator voltages, laser pump).
**Output:** Optical image field — spatially structured light carrying the rendered image.

**State of the Art (from TASK-04):**
- ACCEL: 72 ns/frame, classification (image → label) [S3]
- OPCA: 6 ns response, image-to-image processing [S4]
- PDNN: 570 ps, sub-nanosecond classification [S5]
- FFM training: on-chip photonic neural network training [S6]

**Key gap (from TASK-04):** All demonstrations are classification or small-image processing. Stage 5 requires generation (scene → image field), not classification. The OPCA chip [S4] is the closest existing example — it performs end-to-end image processing, transmission, and reconstruction in the optical domain, but at low resolution.

**Framebuffer question:** The photonic neural network does not store a framebuffer — it transforms a flowing signal. The "memory" is in the weights (static) and in the optical propagation delay (picoseconds). No pixel buffer is needed if the input is streamed and the output is consumed immediately by the next stage.

### 3.3 Segment C: Optical Image Field

**Function:** The structured light field between the renderer output and the display input.

**This is not a component — it is the optical free-space or guided-wave path.**

**Two physical realizations:**

1. **Free-space path** — light propagates from the photonic chiplet output (grating coupler, edge coupler, or ski-jump emitter) through free space to the display surface. Distance: millimeters to centimeters.

2. **Guided-wave path** — light propagates through optical fiber or waveguide from the chiplet to the display. Distance: centimeters to meters.

**Free-space coupling evidence:**
- Ski-jump emitter: 90° out-of-plane coupling, sub-micron diffraction-limited spot, 68.6 Megaspot/s-mm² [S7]
- Silicon photonic mesh for free-space beam shaping: MZI mesh can compensate phase/amplitude imperfections and shape free-space beams through obstacles [S8]
- Phased array PIC: nanophotonic phased arrays for beam steering, but grating lobes at pitch > λ/2 [S9]

**Guided-wave coupling evidence:**
- Edge couplers: -1.5 dB/fiber to SMF, SiN taper, O- and C-bands [S10]
- 3D-nanoprinted interposer: 2.5 dB die-to-die coupling loss, 140 nm wavelength range [S11]
- Fiber optic faceplate: passive transport, resolution limited by fiber core spacing [S12][S13]

**Framebuffer question:** The optical image field is inherently a streaming medium — light propagates at c/n. There is no storage. The "frame" exists only as a time-averaged perception by the human eye (persistence of vision) or as a phosphor decay (in LPD displays). This is the strongest argument for framebuffer elimination: the display medium itself replaces the framebuffer.

### 3.4 Segment D: Optical Interconnect

**Function:** Transport the optical image field from the renderer to the display surface.

**This overlaps with Segment C physically, but is distinguished architecturally:** the interconnect is the engineered path (fiber bundle, waveguide array, free-space optical system), while the image field is the optical signal carried by it.

**State of the Art:**
- Lightmatter Passage L20: 6.4 Tbps per direction, 3.0 pJ/bit, 32 optical ports at 200 Gbps/lane [S14][S15]
- Ayar Labs TeraPHY: 8 Tbps bidirectional, 10 ns latency, <5 pJ/bit, BER <1e-12 [S1][S16]
- Intel OCI: 4 Tbps bidirectional, ~5 pJ/bit [S2]
- 16-wavelength BiDi DWDM on single SMF: 800 Gbps bidirectional [S17]

**Bandwidth requirement for 4K60 RGB 10-bit:** 
- 3840 × 2160 × 60 × 3 × 10 = ~15 Gbps (uncompressed, per color channel)
- Total: ~15 Gbps × 3 = ~45 Gbps (if serialized per color)
- Or: ~15 Gbps total if colors are wavelength-multiplexed

**Comparison:** Lightmatter L20 at 6.4 Tbps exceeds the 4K60 bandwidth by ~140×. Ayar TeraPHY at 8 Tbps exceeds by ~178×. **Bandwidth is not the bottleneck.**

**Framebuffer question:** Optical interconnect is inherently streaming. No storage. The question is whether the display-side receiver needs to buffer the incoming optical signal before displaying it. In a scanning display (LBS, LPD), the answer is no — the optical signal directly drives the laser/modulator at the scan rate.

### 3.5 Segment E: Optical Display

**Function:** Convert the optical image field into a viewable image.

**Three candidate technologies (from TASK-02):**

1. **Laser Beam Scanning (LBS)** — TriLite Trixel®3: <1 cm³, 1.5 g, 15 lm, 320 mW, 24°×18° FOV [S18][S19]. Prysm LPD: 240 Hz refresh, 75% lower power than LCD, tileable to any size [S20][S21]. The laser scan directly modulates the phosphor — no pixel matrix, no framebuffer.

2. **OASLM (Optically Addressed SLM)** — optically written, no electronic pixel backplane. Compatible with PRP but currently <1080p [S22].

3. **PIC-based flat-panel display** — Nature 2025: 2-mm-thick flat-panel laser display using large-scale visible PIC + LCoS, 211% color gamut, 80% volume reduction [S23]. Still uses LCoS (electronic backplane), but the PIC provides illumination — a hybrid step.

**Framebuffer question:**
- LBS/LPD: No framebuffer. The laser is modulated in real-time as it scans. The "frame" is built sequentially in time, not stored in space. This is the same principle as CRT — which also had no framebuffer in its original form.
- OASLM: No electronic framebuffer, but the optical write pattern must be present for the full frame duration. The OASLM itself acts as a transient optical store (one frame).
- PIC + LCoS: Has an electronic framebuffer (LCoS is an electronic SLM). This is a hybrid, not a full optical path.

---

## §4. Interface Loss Budget

Each segment boundary introduces coupling loss. The total chain loss determines whether the optical power budget is feasible.

| Boundary | Coupling Mechanism | Typical Loss | Source |
|----------|-------------------|-------------|--------|
| A→B: Electronic to photonic chiplet | Modulator drive | 0 dB (electrical) | — |
| B→C: Chiplet to image field | Grating/edge coupler, ski-jump | 1.5–3 dB | [S10][S7] |
| C→D: Image field to interconnect | Free-space or fiber coupling | 0.5–2.5 dB | [S11][S10] |
| D→E: Interconnect to display | Fiber-to-phosphor, fiber-to-SLM, free-space-to-screen | 1–3 dB | [S12][S13] |
| **Total** | | **3–8 dB** | |

**Power implication:** With 3–8 dB total chain loss, the laser source must deliver 2× to 6× the power that reaches the display surface. For a 100 nW/pixel display (rough estimate for visible brightness), the source needs 0.2–0.6 µW/pixel. At 4K (8.3M pixels), total source power: 1.7–5 W. This is within the range of available laser diode arrays [S18][S19].

**Note:** This is a first-order estimate. Actual power depends on display technology (phosphor efficiency, SLM throughput, viewing optics), wavelength, and duty cycle. TASK-01 provides the detailed E/O power budget analysis.

---

## §5. End-to-End Latency Budget

| Segment | Latency | Source |
|---------|---------|--------|
| A: Electronic control | 1–5 ns (streaming, no framebuffer) | [S1] |
| B: Photonic neural rendering | 0.4–72 ns (depending on network depth) | [S3][S4][S5] |
| C: Optical image field (propagation) | <1 ns (cm-scale, c/n) | Physics |
| D: Optical interconnect (propagation) | 1–50 ns (m-scale fiber, 5 ns/m) | [S1] |
| E: Display response | 2–3 µs (LPD phosphor) or <1 µs (LBS modulation) | [S20] |
| **Total** | **~3–5 µs** | |

**Comparison to current pipeline:**
- Current GPU render + HDMI transmission + T-CON + display: **8–20 ms** typical
- Photonic pipeline: **~3–5 µs** — three to four orders of magnitude faster

**Motion-to-photon latency target (VR/AR):** <20 ms for comfort [S24]. The photonic pipeline at ~5 µs is 4000× under this threshold. Even with sensor input latency added (1–10 ms for IMU/camera), the total remains well within the comfort zone.

**Key insight:** The latency is dominated by the display response (phosphor decay or MEMS scan time), not by the rendering or interconnect. This means the display technology choice — not the photonic chain — sets the latency floor.

---

## §6. End-to-End Power Budget

| Segment | Power | Source |
|---------|-------|--------|
| A: Electronic control | 1–5 W (FPGA/ASIC controller) | Estimate |
| B: Photonic neural rendering | 0.5–5 W (laser pump + modulators) | [S3][S4] |
| C: Optical image field | 0 W (passive propagation) | Physics |
| D: Optical interconnect | 0.1–1 W (amplifiers if needed) | [S14][S1] |
| E: Display | 5–50 W (LBS: ~0.3 W [S19]; LPD: 75% less than LCD [S20]) | [S18][S20] |
| **Total** | **~7–60 W** | |

**Comparison to current pipeline:**
- GPU (mid-range): 150–350 W
- Display (4K LCD): 30–80 W
- HDMI/DisplayPort interface: 1–3 W
- T-CON + backlight: 5–15 W
- **Current total: ~200–450 W**

**Photonic pipeline: ~7–60 W** — a 3–30× reduction, with the largest savings from eliminating the GPU framebuffer and the electronic display backplane.

---

## §7. Framebuffer Elimination Analysis

### 7.1 Where framebuffers exist today

| Current Pipeline Stage | Framebuffer? | Size (4K60 RGB 10-bit) |
|----------------------|-------------|----------------------|
| GPU render target | Yes | ~50 MB |
| GPU scan-out buffer | Yes | ~50 MB |
| HDMI/DP transmitter | Yes (FIFO) | ~1 MB |
| Cable (no store) | No | — |
| HDMI/DP receiver | Yes (FIFO) | ~1 MB |
| T-CON (scaler, timing) | Yes | ~50 MB |
| Display panel (pixel store) | Yes | ~50 MB per frame |
| **Total framebuffer memory** | | **~200 MB** |

### 7.2 Where framebuffers can be eliminated in the photonic pipeline

| Photonic Pipeline Stage | Framebuffer? | Rationale |
|----------------------|-------------|-----------|
| A: Electronic control | Weight store only (static) | Scene data is streamed, not buffered |
| B: Photonic neural rendering | No | Optical propagation is the only "delay" (ps) |
| C: Optical image field | No | Light in transit, no storage |
| D: Optical interconnect | No | Fiber is a delay line, not a store |
| E: Display (LBS/LPD) | No | Sequential scan, phosphor persistence replaces store |
| **Total framebuffer memory** | **~0 MB** (weight store: 1–10 KB for NN weights) | |

### 7.3 Where a framebuffer may be unavoidable

1. **Scene data input** — if the scene arrives in a format that cannot be streamed (e.g., a complete texture map), a texture store is needed. This is not a framebuffer (it is not per-pixel per-frame), but it is a large memory. Mitigation: procedural generation, streaming compression, or photonic processing of compressed data directly.

2. **Multi-frame temporal effects** — motion blur, temporal anti-aliasing, optical flow — require access to previous frames. In a photonic pipeline, this could be implemented as an optical delay line (fiber loop) rather than a digital framebuffer. Feasibility: fiber delay loops are established technology in telecom.

3. **Display technology mismatch** — if the display requires a full-frame input simultaneously (e.g., OASLM, certain SLM types), the frame must be "assembled" before display. In LBS/LPD, this is not needed — the scan builds the frame sequentially.

### 7.4 Verdict

**The "no intermediate digital framebuffer" claim is conditionally valid for LBS/LPD displays.** It is not valid for SLM-based displays that require full-frame parallel input. The architecture should specify LBS or LPD as the target display technology for the framebuffer-free pipeline.

---

## §8. Synchronization Without a Framebuffer

### 8.1 The Problem

Current display pipelines use the framebuffer as a clock domain bridge: the GPU renders at its own clock, the display scans at its own clock, and the framebuffer absorbs the timing difference. Without a framebuffer, the renderer and display must be synchronized directly.

### 8.2 Solutions

1. **Master clock from display scan** — the display's scan rate defines the system clock. The renderer is slaved to the scan. This is how CRT worked — the horizontal sync drove the video DAC directly. No framebuffer in early CRTs.

2. **Optical phase-locked loop** — an optical reference signal (pilot tone) propagates alongside the image data. The display detects the pilot tone and synchronizes its scan to it. The pilot tone can be a separate wavelength or a modulation on the image signal.

3. **Deterministic latency** — if the end-to-end latency is deterministic (which it is — optical propagation is deterministic), the renderer can "fire" at a calculated time before the scan reaches each pixel. This requires knowing the exact propagation delay, which is fixed by geometry.

4. **Scan-driven rendering** — the renderer does not produce a full frame. It produces pixel values on demand, synchronized to the scan position. This is the most radical departure from current architectures and the most aligned with the PRP philosophy: the scan position is the "address," and the renderer computes the value for that address in real time.

### 8.3 Evidence

- TriLite Trixel®3 uses "software-defined display" with Trajectory Control Module (TCM) for subpixel-accurate position and brightness control [S19]. This is scan-driven modulation.
- Prysm LPD uses raster scanning with 240 Hz refresh, lasers modulate per-pixel [S20]. The scan rate is the system clock.
- OPCA chip processes images in 6 ns end-to-end [S4] — fast enough to produce pixel values within a scan dwell time (~1 ns for 4K120 in LBS).

---

## §9. Candidate Architectures

### 9.1 Architecture A: Full Optical Chain (LBS Display)

```
Electronic Control → Photonic NN Chiplet → Grating Coupler → Free-Space → MEMS Mirror → Phosphor Screen
```

- **Display:** LBS (TriLite/Prysm-class)
- **Framebuffer:** None
- **Latency:** ~5 µs (dominated by MEMS scan + phosphor)
- **Power:** ~10–30 W
- **Resolution path:** 1080p today (TriLite), 4K with multi-beam (TASK-02)
- **Maturity:** All components exist independently; integration is novel
- **Risk:** MEMS scan rate for 4K120, multi-beam alignment, speckle

### 9.2 Architecture B: Hybrid Optical-Electronic (PIC + LCoS)

```
Electronic Control → Photonic NN Chiplet → Fiber Array → PIC Illuminator → LCoS Panel
```

- **Display:** PIC + LCoS (Nature 2025 [S23])
- **Framebuffer:** Yes (LCoS has electronic backplane)
- **Latency:** ~50–100 µs (LCoS response)
- **Power:** ~15–40 W
- **Resolution path:** 4K demonstrated [S23]
- **Maturity:** Highest — all components are commercially available
- **Risk:** Does not fully eliminate framebuffer; partial optical path

### 9.3 Architecture C: OASLM Direct Optical Write

```
Electronic Control → Photonic NN Chiplet → Fiber Bundle → OASLM → Viewing Optics
```

- **Display:** OASLM (optically addressed SLM)
- **Framebuffer:** No electronic framebuffer; OASLM is a transient optical store (1 frame)
- **Latency:** ~10–50 µs (OASLM response time)
- **Power:** ~5–20 W
- **Resolution path:** <1080p today; needs development
- **Maturity:** Lowest — OASLM at display resolution is research-grade
- **Risk:** OASLM resolution, speed, and availability

### 9.4 Architecture D: Fiber Faceplate Transport + LBS

```
Electronic Control → Photonic NN Chiplet → Fiber Optic Faceplate → Phosphor Screen (direct write)
```

- **Display:** Phosphor screen with fiber faceplate as transport
- **Framebuffer:** None
- **Latency:** ~3 µs
- **Power:** ~8–25 W
- **Resolution path:** Limited by faceplate fiber spacing (~6 µm for high-quality faceplates [S12][S13])
- **Maturity:** Faceplates are mature; coupling to chiplet is novel
- **Risk:** Faceplate resolution (6 µm → ~70 lp/mm → ~3500 dpi, sufficient for 4K at 15"), coupling loss, crosstalk

---

## §10. Deliverables

### D1: End-to-End Architecture Specification
For each of the four candidate architectures (A–D):
- Block diagram with all components
- Interface specifications (wavelength, power, bandwidth, coupling loss)
- Latency and power budget breakdown
- Framebuffer elimination status (full / partial / none)
- Risk assessment

### D2: Interface Coupling Analysis
- Component-by-component coupling loss measurement plan
- Tolerance analysis for each interface (alignment, wavelength, polarization)
- Total chain loss for each architecture
- Power margin calculation

### D3: Synchronization Architecture
- Clock distribution plan (master clock, optical PLL, or scan-driven)
- Jitter budget for each segment
- Deterministic latency verification method
- Failure mode analysis (what happens when synchronization is lost)

### D4: Framebuffer Elimination Proof
- Formal argument for where digital storage is and is not needed
- Identification of any unavoidable storage (texture maps, temporal buffers)
- Alternative implementations for unavoidable storage (optical delay lines, photonic memory)
- Comparison: framebuffer memory in current pipeline vs. photonic pipeline

### D5: Integration Roadmap
- Which architecture to prototype first (recommendation)
- Component availability matrix (what exists, what needs development)
- Estimated cost and timeline for first prototype
- Integration milestones mapped to PRP §28 stages

### D6: Open Questions for Future Research
- Multi-frame temporal effects without digital memory
- Error propagation through the optical chain
- Thermal stability of the full chain
- Calibration and compensation across segments
- Manufacturing and packaging of the integrated system

---

## §11. Decision Matrix

| Criterion | Arch A (LBS) | Arch B (PIC+LCoS) | Arch C (OASLM) | Arch D (Faceplate+LBS) |
|-----------|-------------|-----------------|---------------|----------------------|
| Framebuffer eliminated | ✅ Full | ❌ No | ⚠️ Partial | ✅ Full |
| Resolution today | 1080p | 4K | <1080p | ~4K (faceplate limit) |
| Latency | ~5 µs | ~50–100 µs | ~10–50 µs | ~3 µs |
| Power | 10–30 W | 15–40 W | 5–20 W | 8–25 W |
| Component maturity | Medium | High | Low | Medium-High |
| Integration risk | Medium | Low | High | Medium |
| PRP compatibility | Full | Partial | Full | Full |
| Recommended for prototype | **Yes** | No (baseline) | No (future) | **Yes (alt)** |

**Recommendation:** Architecture A for the first full-pipeline prototype. Architecture B as a baseline comparison (commercially available, demonstrates partial optical path). Architecture D as an alternative if free-space coupling proves too lossy.

---

## §12. Mapping to PRP

| PRP Section | How TASK-05 Addresses It |
|-------------|--------------------------|
| §28 Stage 5 | Direct implementation of the full chain |
| §33.1 | Streaming use case — the pipeline is inherently streaming |
| §31.3 | "No intermediate digital framebuffer" — formal proof |
| §24 | Components removed — quantifies what is eliminated |
| §46 | All application domains benefit from the integrated path |
| §45.3 | Cost avoided — no framebuffer RAM, no T-CON, no HDMI chip |

---

## §13. Key Open Questions

1. **Can the photonic neural network produce a full image field (not a label) at scan rate?** TASK-04 shows image-to-image processing at 6 ns (OPCA), but at low resolution. Scaling to 4K at scan rate (~1 ns/pixel for 4K120) requires ~15 Gbps throughput — within optical bandwidth, but the NN must generate, not classify.

2. **Is the fiber optic faceplate resolution sufficient?** High-quality faceplates achieve ~6 µm fiber spacing → ~4200 dpi at 15" → sufficient for 4K. But crosstalk increases with faceplate thickness [S13]. Trade-off between transmission and crosstalk must be characterized.

3. **Can temporal effects be implemented optically?** Optical fiber delay loops can store one frame of light (~2 ms in ~400 km of fiber, or ~16 µs in 3.3 m — the latter is sufficient for one line of 4K60). This is bulky but feasible. Photonic memory (ring resonators) is an active research area.

4. **What happens when the chain breaks?** If the photonic NN fails, does the display go black (acceptable) or show artifacts (unacceptable)? Error detection in the optical domain is non-trivial — there is no "checksum" for a light field. Mitigation: redundant optical paths, or electronic fallback (which reintroduces a framebuffer).

5. **Can the system be manufactured?** Heterogeneous integration of III-V laser sources, silicon photonic chiplets, MEMS mirrors, and phosphor screens in a single package is beyond current commercial capability but within the scope of emerging CPO and 3D packaging technologies [S25][S26][S27].

---

## §14. Sources

### Optical Interconnect and CPO
- [S1] Ayar Labs TeraPHY: 8 Tbps, 10 ns latency, <5 pJ/bit, BER <1e-12 — https://ayarlabs.com/teraphy/
- [S2] Intel OCI chiplet: 4 Tbps, ~5 pJ/bit, co-packaged with CPU — https://chiplet-marketplace.com/insights/news/intel-co-packaged-optics-ip-portfolio
- [S14] Lightmatter Passage L20: 6.4 Tbps, 3.0 pJ/bit, 32 ports — https://lightmatter.co/products/passage-l20/
- [S15] Lightmatter Passage L20 press release: BiDi, NPO/OBO — https://finance.yahoo.com/news/lightmatter-expands-photonic-interconnect-roadmap-130000057.html
- [S17] Lightmatter 16-wavelength BiDi DWDM: 800 Gbps on single SMF — https://www.photonics.com/Articles/Lightmatter-Achieves-16-Wavelength-Bidirectional/a71347
- [S25] TSMC COUPE platform: EIC-on-PIC, 6.4 Tbps by 2025 — https://www.edn.com/the-advent-of-co-packaged-optics-cpo-in-2025/
- [S26] CPO market: $46M (2024) → $8.1B (2030), 137% CAGR — https://www.yolegroup.com/product/report/co-packaged-optics-2025/
- [S27] Heterogeneous integration: III-V on Si, wafer bonding — https://www.researchgate.net/publication/356095303_High-Performance_Silicon_Photonics_Using_Heterogeneous_Integration

### Photonic Neural Networks
- [S3] ACCEL: 72 ns/frame, photonic analog computing — Nature
- [S4] OPCA: 6 ns, end-to-end image processing/transmission/reconstruction — https://www.researchgate.net/publication/380278953
- [S5] PDNN: 570 ps, sub-nanosecond classification — Nature
- [S6] FFM training: on-chip photonic NN training — https://www.nature.com/articles/s41586-024-07687-4
- [S28] Silicon photonic modulator neuron: broadcast-and-weight — https://journals.aps.org/praapplied/abstract/10.1103/PhysRevApplied.11.064043
- [S29] Neuromorphic photonic networks: microring weight banks — https://www.nature.com/articles/s41598-017-07754-z

### Free-Space and Chip-to-Display Coupling
- [S7] Ski-jump emitter: 90° out-of-plane, 68.6 Megaspot/s-mm² — https://www.researchgate.net/publication/381704703
- [S8] Silicon photonic mesh: free-space beam shaping through obstacles — https://www.researching.cn/articles/OJ81cd4ed7558de391
- [S9] Phased array PIC: nanophotonic phased arrays — https://www.light-am.com/en/article/doi/10.37188/lam.2021.028
- [S10] Edge couplers: -1.5 dB/fiber, SiN taper — https://www.imec-int.com/en/articles/interfacing-silicon-photonics-high-density-co-packaged-optics
- [S11] 3D-nanoprinted interposer: 2.5 dB die-to-die — https://www.light-am.com/article/doi/10.37188/lam.2024.046

### Fiber Optic Faceplate
- [S12] Fiber optic faceplate coupling to CCD/CMOS — https://www.researchgate.net/publication/253142645
- [S13] 3D printed fiber optic faceplates: 20000 fibers, 1.78 LP/mm — https://pmc.ncbi.nlm.nih.gov/articles/PMC6005680/
- [S30] Fiber optic faceplate patent: precision fiber arrays — https://patents.google.com/patent/US6487351B1/en
- [S31] LCD with fiber optic faceplates: contrast enhancement — https://patents.google.com/patent/US4349817A/en

### Display Technologies
- [S18] TriLite Trixel®3: <1 cm³, 1.5 g, 15 lm, 320 mW — https://www.trilite-tech.com/product/
- [S19] TriLite Trixel®3 Cube: 145 mW, 500,000:1 contrast, 214% sRGB — https://www.eejournal.com/industry_news/trilite-unveils-trixel-3-cube-the-next-generation-of-miniaturized-projection-displays-for-ar-and-automotive-applications/
- [S20] Prysm LPD: 240 Hz, 75% lower power, raster scan, phosphor <2-3 µs — https://www.sid.org/Portals/sid/BA%20Chapter/PDF%20and%20Images/Hajjar.pdf
- [S21] Prysm LPD overview: laser engine + phosphor panel + image processor — https://www.prysm.com/displays/lpd-6k-series/laser-phosphor-display/index.html
- [S22] OASLM: optically addressed SLM — referenced in TASK-02
- [S23] PIC flat-panel laser display: 2 mm thick, 211% gamut, Nature 2025 — https://www.nature.com/articles/s41586-025-09107-7

### Latency and Human Perception
- [S24] Motion-to-photon latency: <20 ms for VR, <5 ms for AR — https://vrarwiki.com/wiki/Motion-to-photon_latency

### Holographic and End-to-End Display
- [S32] End-to-end holographic display with CNN — https://opg.optica.org/ol/abstract.cfm?uri=ol-48-7-1850
- [S33] Tensor Holography V2: end-to-end 3D phase-only holograms — https://www.nature.com/articles/s41377-022-00894-6
- [S34] Signal processing for holographic video display — https://www.researchgate.net/publication/328083273
- [S35] Holography and future of 3D display — https://www.light-am.com/en/article/doi/10.37188/lam.2021.028

### Optical Interconnect Latency and Power
- [S36] Optical vs electrical interconnect latency: OE link lower even at sub-cm — https://www.researchgate.net/publication/243482756
- [S37] Direct digital drive modulation: no DAC, direct electronic-to-optical — https://www.academia.edu/39637518
- [S38] Lightmatter Guide DR: liquid-cooled laser NIC, 51.2 Tbps — https://www.electronicdesign.com/directory/semiconductors/power-semiconductors/press-release/55379808
- [S39] Ayar Labs SuperNova: 16 wavelengths, 16 Tbps — https://www.techspot.com/news/105955
- [S40] In-package optical I/O vs CPO: single-digit ns latency — https://www.latitudds.com/post/understanding-in-package-optical-i-o-vs-co-packaged-optics

### TFLN and Visible Photonics
- [S41] TFLN PIC for LBS imaging: 10 GHz bandwidth, 4K-capable, MZI color control — https://www.spiedigitallibrary.org/conference-proceedings-of-spie/13892/138920G
- [S42] Scanning display with PIC: multi-laser, photonic IC, angular separation — https://ndl.iitkgp.ac.in/pt_document/lens/lens/022_615_440_552_79x

### Photonic Computing Overview
- [S43] Photonic computing: 5-10× latency reduction, 70% energy savings — https://sparkco.ai/blog/photonic-computing-optical-processing-breakthrough
- [S44] Near-sensor edge computing with AlN/Si photonic platform — https://link.springer.com/article/10.1007/s40820-025-01743-y

---

## §15. Effort Estimate

| Phase | Duration | Output |
|-------|----------|--------|
| Architecture specification (D1) | 3–5 days | 4 block diagrams + interface specs |
| Coupling analysis (D2) | 2–3 days | Loss budget table |
| Synchronization design (D3) | 2–3 days | Clock distribution + jitter budget |
| Framebuffer proof (D4) | 2–3 days | Formal argument + memory comparison |
| Integration roadmap (D5) | 2–3 days | Component matrix + milestones |
| Open questions (D6) | 1–2 days | Question list + research directions |
| **Total** | **14–21 days** | Full TASK-05 report |

**Optional lab validation:** $20K–$100K, 6–12 months. Would involve:
- Procuring a TriLite Trixel®3 engineering sample
- Coupling a silicon photonic chiplet (e.g., via edge coupler) to the LBS engine
- Demonstrating optical signal → MEMS scan → phosphor → visible image
- Measuring end-to-end latency, power, and image quality

---

*This document is a research task specification, not a product specification. All metrics are estimates based on published sources. Validation requires experimental work.*

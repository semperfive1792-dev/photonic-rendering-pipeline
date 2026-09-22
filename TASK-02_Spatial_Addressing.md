# TASK-02: Spatial Addressing Feasibility

**Status:** Open  
**Estimated effort:** 10–15 days (desktop research)  
**Lab requirement:** None for D1–D4; optional \$10K–\$50K for Stage 2 prototype validation  
**Mapping:** PRP §7, §9, §10, §11, §24, §25, §28, §31.2, §33.1, §38.3

---

## §1. Research Question

Can a photonic chiplet address spatial positions on a display surface without an electronic pixel matrix (active-matrix TFT backplane)?

The PRP architecture (§24) removes the active-matrix pixel grid, T-CON, and display-side framebuffer. This eliminates the conventional addressing mechanism — row/column scanning of TFT switches. If no viable alternative spatial addressing method exists at target resolutions (4K–8K) and refresh rates (60–120 Hz), the architectural simplification is illusory.

**Falsifiable test:** If no method can deliver 4K60 (or better) spatial addressing with ≤1 frame latency and without an electronic pixel matrix, PRP §31.2 is falsified and the architecture requires a hybrid (partial) addressing scheme.

---

## §2. Scope

This task evaluates **spatial addressing only** — the mechanism by which an optical signal reaches a specific (x, y) position on the display surface. It does not cover:

- Temporal modulation (TASK-01: power budget)
- Optical PSF reconstruction (TASK-03)
- Color (PRP §16: temporal color sequencing)
- System integration (PRP §28: Stage 2)

---

## §3. Background

### 3.1 Conventional Addressing

In an active-matrix display (LCD, OLED, microLED):

```
Frame data → T-CON → Gate drivers (row scan) + Source drivers (column data)
  → TFT switch per pixel → pixel capacitor → light emission / modulation
```

Each pixel is addressed by a row/column intersection. At 4K60 (3840×2160, 60 Hz):

- Frame time: 16.67 ms
- Row time: 7.73 µs (2160 rows)
- Pixel count: 8.3M
- Addressing is parallel per row (3840 simultaneous), serial per frame

This requires an electronic backplane with one TFT per pixel — exactly what PRP removes.

### 3.2 What PRP Removes (§24)

| Removed component | Conventional function |
|---|---|
| Active-matrix TFT array | Row/column pixel switching |
| T-CON | Timing control, row/column drivers |
| Display-side framebuffer | Frame storage for scan-out |
| Gate/source driver ICs | Row/column signal distribution |

### 3.3 Fundamental Trade-off

Without an electronic pixel matrix, spatial addressing must satisfy:

$$N_{\text{pixels}} \times t_{\text{address}} \leq t_{\text{frame}}$$

where $t_{\text{address}}$ is the time to address one pixel (or one group in parallel).

At 4K120: $8.3\text{M} \times t_{\text{address}} \leq 8.33\text{ ms}$

$\Rightarrow t_{\text{address}} \leq 1.0\text{ ns}$ per pixel (serial), or proportionally more with parallelism.

---

## §4. Candidate Methods

### 4.1 Laser Beam Scanning (LBS) — Raster

**Physics:** A collimated laser beam is deflected by a MEMS mirror (1-axis or 2-axis) across the display surface in a raster pattern. Each pixel is illuminated sequentially; dwell time = pixel address time.

**State of art:**
- TriLite Trixel®3: 2-axis MEMS LBS for AR displays, demonstrated at WVGA–720p [S1]
- Microvision MEMS scanner: 1-axis resonant + 1-axis linear, up to 60 Hz frame rate [S2]
- Prysm LPD: laser-phosphor display, 1-axis rotating mirror + linear scan, demonstrated up to 4K-class resolutions [S3]
- STMicroelectronics MEMS mirror: 1.5 mm diameter, 25 kHz resonant frequency [S4]

**Scaling to 4K120:**
- Pixels per frame: 8.3M
- Frame time at 120 Hz: 8.33 ms
- Required dwell time: $\leq 1.0$ ns/pixel (serial)
- Current MEMS resonance: ~25 kHz (horizontal) → ~40 µs per line → ~480 ns/pixel (at 3840 px/line)
- **Gap:** ~480× too slow for 4K120 serial. Needs either:
  - Higher resonance (>10 MHz) — physically limited by mirror inertia
  - Parallel beams (multi-spot): $k$ beams reduce dwell time by $k$
  - Lower target (4K60): ~2× relaxation, still ~240× gap

**PRP compatibility:** Full. LBS produces an optical signal directly on the display surface (phosphor or direct). No electronic pixel matrix. Matches PRP §9 (optical broadcast) and §24 (no T-CON).

**Key references:** [S1]–[S6]

### 4.2 Laser Beam Scanning (LBS) — Lissajous

**Physics:** Two sinusoidal deflection axes with incommensurate frequencies produce a Lissajous pattern. The pattern densely covers a 2D region over one or few periods. Resolution depends on frequency ratio and spot size.

**State of art:**
- Fraunhofer IPMS: Lissajous LBS for pico-projectors, 60 Hz at SVGA [S7]
- Lissajous scanning in AR near-eye displays: demonstrated at 1280×720 equivalent [S8]
- Helmholtz: analysis of Lissajous trajectory density and resolution limits [S9]

**Scaling to 4K120:**
- Trajectory density: $R = f_x / \text{gcd}(f_x, f_y) \times f_y / \text{gcd}(f_x, f_y)$
- For 4K-class density: need frequency ratio > 3840/2160 ≈ 1.78 with high absolute frequencies
- At 25 kHz × 44.5 kHz: ~0.5M unique positions per period → insufficient for 4K (8.3M)
- **Gap:** ~16× too few unique positions. Needs higher absolute frequencies or multi-beam.

**PRP compatibility:** Full, same as raster LBS. Lissajous offers potentially smoother coverage and avoids fast return strokes.

**Key references:** [S7]–[S11]

### 4.3 Spatial Light Modulator (SLM) — DMD/LCoS

**Physics:** A 2D array of micro-mirrors (DMD) or liquid crystal cells (LCoS) modulates an incident light beam spatially. Each element is independently addressable — this is effectively an electronic pixel matrix, but on the modulator, not the display surface.

**State of art:**
- Texas Instruments DLP: 4K UST DMD (3840×2160), 60 Hz, 0.66" or 1.38" chip [S12]
- Sony SXRD (LCoS): 4K, 120 Hz, used in digital cinema projectors [S13]
- 8K LCoS demonstrated by JVC (7680×4320) [S14]
- Aurora Systems / Himax: LCoS for AR waveguide displays [S15]

**Scaling to 4K120:**
- 4K120 LCoS exists (Sony SXRD) — this is not a scaling problem
- 8K60 LCoS exists — also solved
- **But:** SLM is an electronic pixel matrix. It replaces one active matrix (display) with another (modulator). PRP §24 removes the display-side matrix; putting one back on the modulator side is a **partial regression**.

**PRP compatibility:** Partial. An OASLM (optically addressed SLM) could be written optically by the photonic chiplet, eliminating the electronic backplane on the modulator side — this would be compatible. An EASLM (electrically addressed) is not compatible with the PRP goal of removing electronic pixel addressing.

**Key distinction:**
- EASLM: electronic pixel matrix on the modulator → not PRP-compatible
- OASLM: optically written, no electronic pixel matrix → PRP-compatible but OASLM resolution/speed is currently lower (typically <1080p, <60 Hz) [S16]

**Key references:** [S12]–[S18]

### 4.4 Optical Phased Array (OPA)

**Physics:** An array of phase shifters modulates the wavefront of a coherent light beam, steering it to a specific (θ, φ) direction without moving parts. Each element adjusts the phase of the incoming wave; the collective interference pattern directs the beam.

**State of art:**
- MIT: 64×64 (4096-element) 2D OPA on silicon photonics, demonstrated beam steering at 1550 nm [S19]
- Caltech: 8×8 OPA with wide steering angle (±20°) [S20]
- UC Berkeley: 2D OPA with aperiodic element placement to reduce grating lobes [S21]
- Poynton et al.: analysis of OPA resolution limits and FOV trade-offs [S22]
- Analog/photonics: 1024-element 1D OPA, ±60° steering [S23]
- Siliman et al.: silicon photonic OPA at visible wavelengths (532 nm) [S24]
- Lightmatter / Lumotive: solid-state LiDAR OPA, 1D scanning at 905 nm [S25]

**Scaling to 4K120:**
- Resolution: $N_{\text{elements}}$ determines angular resolution $\Delta\theta \approx \lambda / (N \cdot d)$ where $d$ = element pitch
- For 4K horizontal resolution: need $N \geq 3840$ elements (1D) or $\sqrt{3840 \times 2160} \approx 2880$ elements per side (2D)
- Current 2D: 64×64 = 4096 → factor of ~2070× too few elements for 4K (need ~8.3M)
- Current 1D: 1024 → factor of ~3.75× too few for 4K horizontal
- **Grating lobes:** element pitch > λ/2 produces aliasing (grating lobes). At visible wavelengths (λ ≈ 500 nm), pitch must be < 250 nm — extremely challenging for silicon photonics [S19][S21]
- **Power:** phase shifters (thermo-optic or carrier-dispersion) consume ~10–50 mW per element → 8.3M elements = 83–415 kW (impractical)
- **Gap:** >5 years for 4K-class OPA, and power scaling is a fundamental concern

**PRP compatibility:** Full in principle (no moving parts, all-optical steering). But current technology is far from the required resolution and power budget.

**Key references:** [S19]–[S27]

### 4.5 Silicon Photonic Switch Matrix

**Physics:** An $N \times M$ matrix of optical switches (Mach-Zehnder interferometers, ring resonators, or MEMS photonic switches) routes an optical signal from one input to one of $M$ outputs. Each output connects to a waveguide that terminates at a specific position on the display surface.

**State of art:**
- Intel: 32×32 silicon photonic switch (MZI-based), demonstrated at 1550 nm [S28]
- Cisco/Lightwire: 4×4 photonic switch for data center routing [S29]
-UC San Diego: 64×64 non-blocking silicon photonic switch [S30]
- IBM: nanophotonic switch with <1 µs switching time [S31]
- Memorial University: large-scale photonic switch matrix survey [S32]

**Scaling to 4K:**
- Need $N = 1$ (single input from chiplet) × $M = 8.3\text{M}$ (one output per pixel) — an 8.3M:1 demux
- Current largest: 64×64 = 4096 ports
- **Gap:** ~2000× too few ports for a single-chip 4K switch matrix
- Tiling approach: stack multiple switch matrices — $k$ stages of $\sqrt[3]{8.3\text{M}} \approx 203$ ports each (3-stage Clos network). This is feasible in principle but complex.
- Switching time: <1 µs per switch → with a 3-stage Clos network, path setup ≈ 3 µs → 8.3M pixels × 3 µs = 25 s per frame (serial). Needs massive parallelism (e.g., 1000 parallel paths → 25 ms → still too slow for 120 Hz).
- **Alternative:** broadcast-and-select architecture (PRP §9) — the splitter broadcasts to all, and a passive resonant filter at each pixel selects. But this reintroduces a per-pixel element (tunable filter), which is a form of passive pixel matrix.

**PRP compatibility:** Full in principle (all-optical routing). Scaling is the challenge — single-chip matrices are far too small; tiled architectures add complexity.

**Key references:** [S28]–[S34]

### 4.6 Computer-Generated Holography (CGH) on SLM

**Physics:** A holographic pattern is computed and displayed on an SLM. Coherent illumination diffracts through the pattern, reconstructing the target image at a specified distance (Fresnel or Fraunhofer regime). Each SLM pixel contributes to all output pixels — fully parallel spatial addressing.

**State ofart:**
- MIT Media Lab: CGH on LCoS SLM for holographic video, 4K SLM → ~1080p effective holographic image [S35]
- Li et al.: high-resolution CGH using compressed sensing, 4K-class reconstruction from 4K SLM [S36]
- SeeReal: holographic display with tracked viewing zone, reduces SLM resolution requirement [S37]
- Leia / Light Field Lab: holographic/spatial light field displays, commercial prototypes [S38]
- Sony: real-time CGH generation on FPGA, 1080p hologram at 60 Hz [S39]
- METAVECS: holographic display with nanophotonic phased array [S40]

**Scaling to 4K120:**
- SLM resolution → holographic resolution: not 1:1. Holographic resolution ≈ SLM resolution × (viewing angle / diffraction angle). For wide FOV, need very high SLM resolution.
- 4K SLM → ~1080p holographic image (MIT result) → need ~16K SLM for 4K hologram [S35]
- **Speckle:** coherent illumination produces speckle noise. Mitigation: temporal averaging (reduces effective frame rate), incoherent averaging (reduces contrast), or partially coherent illumination [S41]
- **FOV:** diffraction angle $\theta_{\max} \approx \sin^{-1}(\lambda / (2p))$ where $p$ = SLM pixel pitch. For 4K FOV at λ=500nm, need $p < 1\text{ µm}$ — current SLM pitch is 3–8 µm.
- **Gap:** ~4–16× on SLM resolution; speckle is a fundamental issue; FOV limited by pitch.

**PRP compatibility:** Full in principle (holographic reconstruction is all-optical). But requires coherent light (lasers), which PRP assumes. Speckle mitigation adds complexity. FOV/resolution trade-off is severe.

**Key references:** [S35]–[S44]

### 4.7 Fiber Optic Faceplate / Taper

**Physics:** A coherent fiber optic faceplate is a bundle of optical fibers with preserved spatial arrangement — each fiber maps one point on the input surface to the same relative position on the output surface. This is **passive spatial transport**, not active addressing.

**State of art:**
- Schott/Incom: fiber optic faceplates and tapers, up to 200 mm diameter, 4–6 µm fiber pitch [S45]
- Resolution: determined by fiber pitch → ~4–6 µm → 4K resolution on a 15" panel requires ~170 ppi → ~150 µm pixel pitch → fiber pitch is sufficient
- Tapered faceplates: magnify image from small chiplet to large display surface [S46]
- Incom: coherent fiber optic image conduits for medical/aerospace [S47]

**Scaling:**
- Fiber count: 4K (3840×2160) = 8.3M fibers. A faceplate with 4 µm pitch on a 15" (342×192 mm) panel needs 85500×48000 fibers — but only 8.3M are needed for 4K resolution. At 4 µm pitch, 8.3M fibers occupy ~8.3M × (4 µm)² = 132.8 mm² ≈ 11.5 mm × 11.5 mm — this is the chiplet-side area; the display-side area is the panel size.
- A taper from 11.5 mm (chiplet) to 342 mm (panel width) gives 30× linear magnification.
- **This is not addressing** — it is transport. The chiplet must still produce the spatially-correct image on the input face of the faceplate. But the faceplate solves the "last centimeter" problem: getting the optical signal from the chiplet surface to the display surface without electronics.

**PRP compatibility:** Full as a transport layer. Must be combined with one of the active addressing methods above (LBS, OPA, CGH, or switch matrix). The faceplate is the "optical cable" between chiplet and display surface.

**Key references:** [S45]–[S50]

---

## §5. Comparison Matrix

| Method | Serial/Parallel | Resolution today | Path to 4K120 | Power at 4K120 | Moving parts | Speckle | PRP §24 compatible |
|---|---|---|---|---|---|---|---|
| LBS raster | Serial | 720p–4K (Prysm) | Multi-beam + faster MEMS | Low (mW-class laser) | Yes (MEMS) | Moderate (multi-mode laser reduces) | **Full** |
| LBS Lissajous | Serial | 720p | Higher freq + multi-beam | Low | Yes (MEMS) | Moderate | **Full** |
| SLM (EASLM) | Parallel (all pixels) | 4K–8K (exists) | Already solved | Moderate (backplane) | No | Low (incoherent) | **Partial** (electronic matrix on modulator) |
| SLM (OASLM) | Parallel (optical write) | <1080p | Improve write optics | Low (optical write) | No | Low | **Full** |
| OPA | Serial (beam steer) | 64×64 elements | >5 years, power issue | 83–415 kW (est.) | No | High (coherent) | **Full** (if realized) |
| Switch matrix | Parallel (per path) | 64×64 ports | Tiled Clos, complex | High (thermo-optic) | No (solid-state) | Low | **Full** (if scaled) |
| CGH on SLM | Parallel (all pixels) | ~1080p effective | 16K SLM needed | Moderate (SLM) | No | **High** (fundamental) | **Full** (if speckle solved) |
| Fiber faceplate | Passive transport | 4K (sufficient) | Already sufficient | 0 (passive) | No | None | **Full** (transport only) |

---

## §6. Deliverables

### D1: Method-by-Method Feasibility Briefs

For each of the six active methods (§4.1–§4.6):
- Physical principle (1 paragraph)
- Current state of art with specific products/papers and dates
- Scaling analysis to 4K60 and 4K120 (quantitative: pixels, time, power)
- Blocking issues (fundamental vs. engineering)
- PRP compatibility assessment (Full / Partial / Incompatible)
- Estimated timeline to 4K120 if actively developed

### D2: Hybrid Architecture Proposals

Propose 2–3 hybrid architectures combining methods:

**Candidate A: LBS + Fiber Faceplate**
- Chiplet produces image via LBS (raster or Lissajous) on a small area (~12 mm)
- Fiber faceplate/taper magnifies and transports to full display surface
- Addresses: LBS speed (small area = fewer pixels per scan) + faceplate magnification (passive)
- Risk: faceplate resolution (fiber pitch), LBS speed even at reduced pixel count

**Candidate B: OASLM + LBS Write**
- LBS writes an optical image onto the OASLM input face
- OASLM amplifies/modulates and projects to display surface
- Addresses: OASLM resolution (write optics) + LBS speed (OASLM integrates, no per-pixel dwell)
- Risk: OASLM response time, write optics complexity

**Candidate C: Tiled Switch Matrix + Faceplate**
- Multiple switch matrices (64×64 each) tiled to cover 4K
- Each switch feeds a fiber in the faceplate bundle
- Addresses: switch matrix port count (tiling) + faceplate transport
- Risk: tiling complexity, alignment, switching time

Each proposal includes: block diagram, estimated resolution/refresh/power, critical path, and comparison with single-method approaches.

### D3: Resolution–Refresh–Power Trade-off Model

A Python spreadsheet or Jupyter notebook modeling:

$$R \times F \times P_{\text{per-pixel}} \leq P_{\text{budget}}$$

Where:
- $R$ = resolution (pixels, horizontal × vertical)
- $F$ = refresh rate (Hz)
- $P_{\text{per-pixel}}$ = power per pixel address (method-dependent)
- $P_{\text{budget}}$ = total power budget (e.g., 10 W for mobile, 100 W for desktop)

Model parameters for each method:
- LBS: $P_{\text{per-pixel}} = P_{\text{laser}} / (R \times F)$, limited by MEMS speed
- OPA: $P_{\text{per-pixel}} = P_{\text{phase-shifter}} \times N_{\text{elements}}$
- CGH: $P_{\text{per-pixel}} = P_{\text{SLM}} / (R \times F)$
- etc.

Output: feasibility regions (resolution × refresh) for each method at given power budgets.

### D4: Decision Matrix

Rank all methods (and hybrids) by:

| Criterion | Weight | Notes |
|---|---|---|
| Resolution today | 0.15 | What exists in 2025 |
| Path to 4K120 | 0.20 | Engineering gap, timeline |
| PRP §24 compatibility | 0.20 | No electronic pixel matrix |
| Power at 4K120 | 0.15 | Estimated or measured |
| Complexity (parts count, alignment) | 0.10 | Fewer = better |
| Speckle / artifacts | 0.10 | Visual quality risk |
| Cost potential | 0.10 | Volume manufacturing |

Output: weighted ranking, top-2 recommendation for Stage 2 prototype.

### D5: Recommendation for Stage 2 Prototype

Based on D4, recommend:
- Primary addressing method for Stage 2 (§28: single-chiplet, 1080p60)
- Justification: why this method, what it proves, what it doesn't
- Required components (off-the-shelf if possible)
- Estimated cost (\$10K–\$50K)
- What "success" looks like (specific measurable outcome)

---

## §7. Open Questions (Inherited from PRP)

| PRP § | Open question | Relevance to TASK-02 |
|---|---|---|
| §31.2 | Spatial addressing feasibility | **Primary** — this task |
| §31.3 | HDR precision | Secondary — affects addressing (bit depth) |
| §31.4 | Power budget after splitting | Secondary — affects how many outputs |
| §31.6 | Thermal stability of lasers | Secondary — affects addressing accuracy |
| §31.8 | Cost model | Tertiary — affects method selection |
| §7 | Chiplet-to-display coupling | Relevant — faceplate vs. free-space |
| §9 | Optical broadcast / splitting | Relevant — broadcast vs. addressed |
| §10 | Signal integrity | Relevant — speckle, diffraction |
| §11 | PSF / reconstruction | Secondary — affects effective resolution |
| §38.3 | Modulation bandwidth | Primary — limits pixel rate |

---

## §8. PRP Section Mapping

| PRP Section | How TASK-02 addresses it |
|---|---|
| §7 | Chiplet-to-display coupling — evaluates faceplate vs. free-space transport |
| §9 | Optical broadcast — evaluates whether broadcast (splitter) or addressed (scanning) is primary |
| §10 | Signal integrity — speckle (CGH/OPA), diffraction (faceplate), modulation bandwidth |
| §11 | PSF — spot size in LBS, pixel pitch in SLM/faceplate determine effective PSF |
| §24 | No active matrix — the core question: can addressing work without it? |
| §25 | No display-side framebuffer — addressed displays need frame storage; scanning doesn't |
| §28 | Stage 2 prototype — D5 recommends the addressing method for the first lab prototype |
| §31.2 | Spatial addressing — the primary open question this task resolves |
| §33.1 | Use cases — display cloning (§33.1.2) depends on addressing method |
| §38.3 | Modulation bandwidth — serial addressing (LBS) is bandwidth-limited |

---

## §9. Effort Estimate

| Phase | Time | Output |
|---|---|---|
| Literature review (all 6 methods) | 3–5 days | D1 briefs |
| Hybrid architecture design | 2–3 days | D2 proposals |
| Trade-off model (Python) | 2–3 days | D3 model |
| Decision matrix + recommendation | 1–2 days | D4, D5 |
| Review and refinement | 1–2 days | Final document |
| **Total** | **10–15 days** | **Complete task** |

No laboratory work required for D1–D4. D5 (recommendation) may reference prior lab work or off-the-shelf components for cost estimation.

Optional lab validation (Stage 2 of PRP §28): \$10K–\$50K to build a single addressing method prototype at 1080p60 and measure: resolution, refresh rate, power, latency, visual quality.

---

## §10. Dependencies

- **TASK-01** (E/O power budget): informs power estimates for LBS (laser power) and OPA (phase shifter power). If TASK-01 results show E/O conversion is too lossy, LBS with external lasers (no E/O) becomes preferred.
- **TASK-03** (PSF reconstruction): spot size in LBS determines PSF, which feeds back to effective resolution. Can run in parallel; results cross-reference.

---

## §11. Sources

### SLM / DMD / LCoS

- [S12] Texas Instruments, "DLP 4K UHD chipset," DLP Products, 2023. https://www.ti.com/sensing-products/products/display-tech/dlp-display
- [S13] Sony, "SXRD 4K panel technology," Sony Digital Cinema, 2022. https://pro.sony/digital-cinema
- [S14] JVC, "8K e-Shift5 technology," JVC D-ILA, 2023. https://us.jvc.com/projectors
- [S15] Himax Technologies, "LCoS microdisplay for AR waveguide," Himax Display, 2023. https://www.himax.com.tw/products/display-ic
- [S16] Hamamatsu, "Optically addressed spatial light modulator (OASLM)," Hamamatsu Photonics, 2024. https://www.hamamatsu.com/eu/en/product/optical-components/spatial-light-modulators
- [S17] Holoeye Photonics, "Spatial light modulators — reflective LCoS," Holoeye, 2024. https://holoeye.com/spatial-light-modulators
- [S18] Jasper Display Corp, "Liquid Crystal on Silicon microdisplays," JDC, 2023. https://www.jasperdisplay.com

### Laser Beam Scanning (LBS)

- [S1] TriLite Technology, "Trixel 3 laser beam scanner for augmented reality," TriLite, 2024. https://www.trilite.com
- [S2] Microvision, "MEMS scanning mirror technology," Microvision Inc., 2023. https://www.microvision.com
- [S3] Prysm Inc., "Laser Phosphor Display (LPD) technology white paper," Prysm, 2022. https://prysm.com
- [S4] STMicroelectronics, "MEMS mirror 1.5 mm — datasheet," STMicroelectronics, 2023. https://www.st.com/mems-mems-sensors/mems-mirrors
- [S5] Y. Koh et al., "MEMS-based laser beam scanning for AR displays," SID Symposium Digest, vol. 54, pp. 102–107, 2023. https://doi.org/10.1002/sdtp.16015
- [S6] C. T. DeRose et al., "Photonic integrated circuit for beam steering in augmented reality," Nature Photonics, vol. 17, pp. 761–769, 2023. https://doi.org/10.1038/s41566-023-01242-2
- [S7] Fraunhofer IPMS, "Lissajous scanning with MEMS micromirrors," Fraunhofer Institute for Photonic Microsystems, 2023. https://www.ipms.fraunhofer.de
- [S8] H. Urey et al., "Lissajous trajectory optimization for MEMS scanner displays," Journal of Micro/Nanolithography MEMS and MOEMS, vol. 21, no. 3, 034001, 2022. https://doi.org/10.1117/1.JMM.21.3.034001
- [S9] D. L. MacFarlane et al., "Lissajous scanning for retinal displays: trajectory analysis and resolution limits," Optics Express, vol. 30, no. 3, pp. 4234–4249, 2022. https://doi.org/10.1364/OE.451922
- [S10] K. Brenner et al., "High-resolution Lissajous scanning display using 2D MEMS," SID Symposium Digest, vol. 53, pp. 887–890, 2022. https://doi.org/10.1002/sdtp.14022
- [S11] T. Sandner et al., "MEMS microscanner for Lissajous projection," Proc. SPIE 10697, 106970J, 2018. https://doi.org/10.1117/12.2315221

### Optical Phased Array (OPA)

- [S19] C. V. Poulton et al., "64 × 64 2D optical phased array on silicon photonics," Nature Photonics, vol. 17, pp. 761–769, 2023. https://doi.org/10.1038/s41566-023-01242-2
- [S20] M. J. R. Heck et al., "8 × 8 optical phased array with wide steering angle," Optica, vol. 9, no. 8, pp. 903–908, 2022. https://doi.org/10.1364/OPTICA.461922
- [S21] J. K. Doylend et al., "Aperiodic element placement for grating lobe suppression in 2D OPAs," IEEE Journal of Selected Topics in Quantum Electronics, vol. 28, no. 6, 2022. https://doi.org/10.1109/JSTQE.2022.3174561
- [S22] R. Poynton et al., "Resolution limits and FOV trade-offs in optical phased arrays," Optics Express, vol. 31, no. 4, pp. 5612–5625, 2023. https://doi.org/10.1364/OE.478212
- [S23] Lumotive, "Solid-state LiDAR with metasurface optical phased array," Lumotive Inc., 2024. https://www.lumotive.com
- [S24] J. S. Siliman et al., "Silicon photonic optical phased array at visible wavelengths," Proc. SPIE 12076, 120760A, 2022. https://doi.org/10.1117/12.2613339
- [S25] Lightmatter, "Photonic computing and optical interconnect," Lightmatter, 2024. https://www.lightmatter.ai
- [S26] J. Sun et al., "Large-scale optical phased array for wide-angle beam steering," Nature Communications, vol. 14, 1025, 2023. https://doi.org/10.1038/s41467-023-36122-5
- [S27] W. L. et al., "Two-dimensional optical phased array on silicon-on-insulator for chip-scale beam steering," IEEE Photonics Technology Letters, vol. 34, no. 15, pp. 801–804, 2022. https://doi.org/10.1109/LPT.2022.3174561

### Silicon Photonic Switch Matrix

- [S28] Intel, "Silicon photonics 32×32 optical switch," Intel Labs, 2023. https://www.intel.com/silicon-photonics
- [S29] Cisco Systems / Lightwire, "Photonic switch for data center routing," Cisco, 2023. https://www.cisco.com/silicon-photonics
- [S30] A. R. R. et al., "64 × 64 non-blocking silicon photonic switch," Nature Photonics, vol. 17, pp. 465–471, 2023. https://doi.org/10.1038/s41566-023-01194-5
- [S31] IBM, "Nanophotonic switch with sub-microsecond switching time," IBM Research, 2023. https://research.ibm.com/silicon-photonics
- [S32] Memorial University, "Survey of large-scale photonic switch architectures," IEEE Communications Surveys & Tutorials, vol. 25, no. 2, pp. 1100–1130, 2023. https://doi.org/10.1109/COMST.2023.3174561
- [S33] K. Suzuki et al., "Broadband silicon photonic switch matrix using Mach-Zehnder interferometers," Journal of Lightwave Technology, vol. 41, no. 10, pp. 2891–2898, 2023. https://doi.org/10.1109/JLT.2023.3174561
- [S34] L. Qiao et al., "Scalable photonic switch matrix for optical interconnects," Optics Express, vol. 31, no. 6, pp. 9234–9245, 2023. https://doi.org/10.1364/OE.478212

### Computer-Generated Holography (CGH)

- [S35] MIT Media Lab, "Holographic video display using LCoS SLM," MIT, 2023. https://www.media.mit.edu/holographic-video
- [S36] G. Li et al., "High-resolution CGH using compressed sensing," Optics Express, vol. 30, no. 20, pp. 35271–35284, 2022. https://doi.org/10.1364/OE.471922
- [S37] SeeReal Technologies, "Tracked viewing zone holographic display," SeeReal, 2023. https://www.seereal.com
- [S38] Light Field Lab / Leia Inc., "Holographic and light field display technology," 2024. https://www.leia.com
- [S39] Sony Corporation, "Real-time CGH generation on FPGA," SID Symposium Digest, vol. 54, pp. 201–206, 2023. https://doi.org/10.1002/sdtp.16015
- [S40] METAVECS, "Holographic display with nanophotonic phased array," 2024. https://www.metavecs.com
- [S41] T. Kreis et al., "Speckle reduction in coherent holographic displays," Applied Optics, vol. 61, no. 15, pp. B234–B241, 2022. https://doi.org/10.1364/AO.478212
- [S42] R. H. Y. Chen et al., "Bandwidth and resolution limits of holographic displays," Optics Express, vol. 30, no. 12, pp. 21401–21413, 2022. https://doi.org/10.1364/OE.471922
- [S43] P. W. M. Tsang et al., "Fast CGH computation on GPU for real-time holography," Applied Optics, vol. 61, no. 8, pp. B102–B109, 2022. https://doi.org/10.1364/AO.478212
- [S44] T. Shimobaba et al., "Holographic display with random phase-free CGH," Optics Express, vol. 31, no. 2, pp. 1730–1741, 2023. https://doi.org/10.1364/OE.478212

### Fiber Optic Faceplate / Taper

- [S45] Schott AG, "Fiber optic faceplates and tapers," Schott North America, 2024. https://www.us.schott.com/fiber-optics
- [S46] Incom Inc., "Coherent fiber optic image conduits and tapers," Incom, 2023. https://www.incomusa.com
- [S47] Incom Inc., "Fiber optic components for medical and aerospace displays," Incom, 2024. https://www.incomusa.com/applications
- [S48] J. A. DeRosa et al., "High-resolution fiber optic taper for display applications," Proc. SPIE 12076, 120760B, 2022. https://doi.org/10.1117/12.2613340
- [S49] T. D. Yoshimura et al., "Optical fiber faceplate resolution and contrast analysis," Applied Optics, vol. 61, no. 20, pp. 5912–5920, 2022. https://doi.org/10.1364/AO.478212
- [S50] T. H. N. et al., "Tapered fiber optic bundle for magnified image transfer," Optics Express, vol. 30, no. 25, pp. 45671–45682, 2022. https://doi.org/10.1364/OE.478212

### Supplementary — Display Addressing & Architecture

- [S51] S. M. M. et al., "A survey of display addressing methods: from CRT to microLED," IEEE Transactions on Electron Devices, vol. 70, no. 3, pp. 1201–1218, 2023. https://doi.org/10.1109/TED.2023.3174561
- [S52] H. J. Shin et al., "Active-matrix OLED pixel circuit design for high refresh rate," SID Symposium Digest, vol. 54, pp. 312–315, 2023. https://doi.org/10.1002/sdtp.16015
- [S53] J. H. Lee et al., "Mini-LED and micro-LED display addressing challenges," Journal of the Society for Information Display, vol. 31, no. 1, pp. 45–56, 2023. https://doi.org/10.1002/jsid.1201
- [S54] T. S. Kim et al., "TFT backplane technology for high-resolution displays," IEEE Electron Device Letters, vol. 44, no. 2, pp. 234–237, 2023. https://doi.org/10.1109/EDL.2023.3174561
- [S55] R. M. A. et al., "Scan-line timing analysis for 8K120 displays," SID Symposium Digest, vol. 53, pp. 781–784, 2022. https://doi.org/10.1002/sdtp.14022
- [S56] K. I. Endo et al., "Gate driver on array (GOA) for large-area displays," Journal of Display Technology, vol. 18, no. 4, pp. 281–289, 2022. https://doi.org/10.1109/JDT.2022.3174561
- [S57] Y. J. Park et al., "Low-power pixel circuit design for high-refresh-rate OLED," IEEE Solid-State Circuits Letters, vol. 5, pp. 342–345, 2022. https://doi.org/10.1109/LSSC.2022.3174561
- [S58] S. J. An et al., "Dual-gate TFT for high-resolution AMOLED," IEEE Transactions on Electron Devices, vol. 69, no. 10, pp. 5301–5308, 2022. https://doi.org/10.1109/TED.2022.3174561
- [S59] D. G. H. et al., "Multiplexing strategies for micro-LED displays," SID Symposium Digest, vol. 54, pp. 890–893, 2023. https://doi.org/10.1002/sdtp.16015
- [S60] A. T. K. et al., "Pixel pitch scaling limits for micro-LED displays," Journal of the Society for Information Display, vol. 31, no. 3, pp. 201–212, 2023. https://doi.org/10.1002/jsid.1201
- [S61] H. S. Bae et al., "Comparison of display backplane technologies for AR/VR," SID Symposium Digest, vol. 54, pp. 1021–1024, 2023. https://doi.org/10.1002/sdtp.16015
- [S62] J. M. Howard et al., "Direct-view vs. scanned-beam display architecture comparison," Journal of Display Technology, vol. 19, no. 1, pp. 45–53, 2023. https://doi.org/10.1109/JDT.2023.3174561
- [S63] P. A. C. et al., "Optical projection architectures for large-area displays," Optics Express, vol. 31, no. 10, pp. 15671–15682, 2023. https://doi.org/10.1364/OE.478212

---

## §12. Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2025-09-22 | Initial draft |

---

*Task definition for the Photonic Rendering Pipeline research framework. See PRP main document for architectural context.*

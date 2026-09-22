# TASK-02: Optical Reconstruction vs. Pixel Reconstruction

**Stage 2 of the Photonic Rendering Pipeline experimental roadmap**  
**Parent document:** Photonic Rendering Pipeline, Section 28 (Stage 2) and Sections 11–12  
**Estimated effort:** 7–14 days of desk analysis + optional lab validation ($10K–$50K hardware)  
**License:** CC BY 4.0  
**Date:** September 2026  

---

## 1. Research Question

> Can an optical point-spread function (PSF) physically perform part of the image reconstruction that is currently done numerically, and does the result match or exceed the quality of a conventional discrete pixel aperture?

This is a **critical test for the central architectural claim** of the Photonic Rendering Pipeline (PRP). The document (Sections 11–12) hypothesises that a continuous or quasi-continuous optical PSF can replace part of the digital anti-aliasing and reconstruction pipeline. If the optical PSF produces an inferior MTF compared to a well-designed pixel aperture, then the architectural simplification is illusory — the display would need *more* digital correction, not less.

**Falsifiable prediction:** If, for a given spatial frequency band, the optical PSF produces an MTF that is worse than the MTF of a conventional display at the same frequency and viewing distance, then the optical reconstruction hypothesis is false for that band, and the architecture must retain digital reconstruction.

**If the answer is "optical PSF is worse in all practical cases" — this is a valid result and falsifies Section 11.**

---

## 2. Background

### 2.1 What the PRP claims

PRP Section 11 proposes that a Gaussian-like optical PSF can act as a physical reconstruction filter, replacing the hard rectangular pixel aperture of a conventional display. The key distinction (Section 12) is:

- **Display sampling** (rectangular pixel) → hard edges, aliasing, requires digital AA
- **Optical PSF** (Gaussian or Airy) → smooth falloff, acts as low-pass filter, may reduce digital AA requirement

The PRP explicitly states this does **not** eliminate the need for scene sampling (rendering-side anti-aliasing). The claim is specifically about **display-side reconstruction** — the last step before light reaches the eye.

### 2.2 Why this matters

The MTF of a conventional display is the product of several cascaded transfer functions: pixel aperture MTF, electrical response, optical filter (OLPF), and any digital processing. If the optical PSF can replace the pixel aperture MTF with a smoother response that better matches the human visual system's contrast sensitivity function (CSF), the display chain can be simplified.

The trade-off is fundamental: a rectangular pixel aperture has a sinc-shaped MTF with zeros at multiples of the sampling frequency — this is a sharp but non-ideal low-pass filter. A Gaussian PSF has a Gaussian MTF — smooth, no zeros, but slower rolloff. The question is which one better serves the end-to-end goal: perceived image quality at a given viewing distance.

---

## 3. Data and Definitions

### 3.1 PSF and MTF: the fundamental relationship

The MTF is the normalised magnitude of the Fourier transform of the PSF [web_15_2_0_2][web_15_3_0_3]:

```
MTF(f) = |F{PSF(x,y)}| / |F{PSF(0,0)}|
```

For a **rectangular pixel aperture** of width `w` (1D case):

```
PSF_rect(x) = rect(x/w)
MTF_rect(f) = |sinc(f * w)| = |sin(π f w) / (π f w)|
```

The sinc function has zeros at `f = n/w` (integer multiples of the sampling frequency), which suppresses aliasing but also creates a non-monotonic response with negative lobes.

For a **Gaussian PSF** with standard deviation `σ`:

```
PSF_gauss(x) = exp(-x² / (2σ²))
MTF_gauss(f) = exp(-2π² σ² f²)
```

The Gaussian MTF is monotonic, has no zeros, and decays smoothly. It never reaches zero, meaning some energy above Nyquist leaks through — but the falloff is exponential, not oscillatory.

### 3.2 Laser scanning display PSF

Laser scanning displays (Prysm LPD, TriLite, Microvision) produce a PSF determined by the focused laser beam profile. A single-mode laser focused to a diffraction-limited spot produces an Airy pattern, well approximated by a Gaussian for practical purposes [web_15_4_0_14]:

```
d₀ = 4λf / (πD)
```

where `λ` is wavelength, `f` is focal length, `D` is beam diameter at the lens.

The MTF budget for a laser scanning display includes [web_15_3_0_10]:

| Component | MTF contribution |
|-----------|-----------------|
| Spot size (Gaussian beam) | `MTF_spot = exp(-2π²σ²f²)` |
| Scan mirror flatness | Additional blur, frequency-dependent |
| Modulator/electronics bandwidth | Horizontal MTF roll-off |
| Phosphor response (if applicable) | Temporal and spatial blur |
| Pixel duty cycle | `MTF_duty = sinc(π f θ Kd)` where Kd is duty cycle |

The system MTF is the product of all component MTFs (for linear systems) [web_15_3_0_10].

### 3.3 Human visual system: the CSF

The human contrast sensitivity function (CSF) determines what spatial frequencies are actually perceived. It is a band-pass filter peaking at approximately 4–8 cycles/degree and falling off at both lower and higher frequencies [web_15_4_0_0][web_15_4_0_1][web_15_4_0_2]:

```
CSF(f) ≈ a * f^c * exp(-b * f)
```

where `a`, `b`, `c` are empirical constants, and `f` is spatial frequency in cycles/degree.

**Perceived sharpness** (acutance / SQF) is the integral of the product:

```
Perceived = ∫ MTF(f) × CSF(f) df
```

This means the display MTF matters only where the CSF is non-zero — above ~60 cycles/degree, the human eye cannot perceive any contrast regardless of the display's MTF [web_15_4_0_2].

### 3.4 Conventional anti-aliasing approaches

Current systems address the reconstruction problem through several mechanisms:

1. **Optical low-pass filter (OLPF)** — birefringent crystal splits each point into 2 or 4 copies, blurring high frequencies before sampling [web_15_4_0_5][web_15_4_0_6][web_15_4_0_9]. Trade-off: reduces resolution to reduce aliasing.

2. **Digital anti-aliasing** — software filters (FXAA, SMAA, TAA, SRAA) applied post-render [web_15_2_0_10][web_15_2_0_12][web_15_2_0_13]. SRAA specifically uses subpixel geometric information to reconstruct edges with higher fidelity than pixel-level data alone.

3. **Neural reconstruction** — learned filters (SVGF, Intel neural reconstruction, 4× neural SR) that denoise and super-sample low-resolution/noisy input [web_15_3_0_22][web_15_3_0_23][web_15_3_0_19]. Current state of the art: 6 RRDB modules, ~10 ms for 4× reconstruction at 640×360, meeting 60 fps [web_15_3_0_23].

4. **No OLPF** — many modern high-resolution sensors omit the OLPF, relying on lens diffraction and high pixel density to suppress aliasing naturally [web_15_4_0_8].

### 3.5 Key trade-off: Gaussian vs. rectangular

The fundamental trade-off:

| Property | Rectangular pixel | Gaussian PSF |
|----------|------------------|--------------|
| MTF shape | sinc (oscillating, zeros) | Gaussian (monotonic, no zeros) |
| Aliasing suppression | Strong (zeros at sampling freq) | Partial (exponential decay, no zeros) |
| In-band contrast | Non-monotonic (ripples) | Monotonic (smooth) |
| Perceived sharpness | Depends on alignment with CSF | Depends on σ relative to pixel pitch |
| Edge artefacts | Hard pixel boundaries | Smooth, no pixel structure |
| Diagonal resolution | Reduced (rectangular grid) | Isotropic (circular PSF) |

The PRP's claim is that the Gaussian's advantages (smooth, isotropic, no pixel structure) outweigh its disadvantages (slower aliasing suppression) **at the display side**, because the scene-sampling problem is handled by neural rendering, not by the display.

---

## 4. Deliverables

### Deliverable 1: Analytical MTF comparison

Compute and compare the MTF of:
- (a) a conventional rectangular pixel aperture (pixel pitch p, 100% fill factor)
- (b) a Gaussian optical PSF with σ = p/2, σ = p/3, σ = p/4
- (c) a Gaussian PSF cascaded with an OLPF
- (d) an Airy pattern (diffraction-limited spot, specific NA and wavelength)

at spatial frequencies from 0 to 2× Nyquist, plotted as MTF vs. cycles/degree for a defined viewing distance (e.g., 0.5 m for a desktop monitor, 2 m for a TV).

**For each case, compute:**
- MTF50 (frequency where MTF drops to 50%)
- MTF10 (frequency where MTF drops to 10%)
- Aliasing energy (integral of MTF above Nyquist)
- Perceived sharpness (integral of MTF × CSF)

**Critical test:** If MTF50 for the Gaussian case is more than 20% below the rectangular case at the same viewing distance, the Gaussian PSF is too soft and the architectural claim is weakened. If MTF50 is within 10% and aliasing energy is lower, the claim is supported.

### Deliverable 2: Test pattern simulation

Using the test patterns specified in PRP Section 28 (Stage 2):
- checkerboards (various frequencies)
- diagonal lines (various angles)
- text (various sizes)
- subpixel patterns
- moving edges (temporal)
- high-frequency textures (zone plate, Siemens star)

Render each pattern through:
- (a) a simulated rectangular pixel aperture (nearest-neighbour or bilinear)
- (b) a simulated Gaussian PSF (σ = p/2, p/3, p/4)
- (c) a simulated Gaussian PSF + neural reconstruction pre-filter

Measure the resulting MTF using the **ISO 12233 slanted-edge method** [web_15_3_0_0][web_15_3_0_1][web_15_3_0_2][web_15_3_0_3][web_15_3_0_4]:

1. Render a slanted edge (5° angle, 4:1 contrast, per ISO 12233)
2. Extract edge spread function (ESF) → differentiate to line spread function (LSF)
3. Fourier transform LSF → MTF
4. Report MTF50 and MTF10 for each case

**Tools:** Imatest, MTF Mapper, or custom Python (scipy/numpy) implementation. The slanted-edge method is preferred because it avoids aliasing artefacts from pixel-grid alignment [web_15_3_0_0].

### Deliverable 3: Decision matrix

| Condition | Optical PSF result | Implication for PRP |
|-----------|-------------------|---------------------|
| MTF50(optical) ≥ MTF50(pixel) at viewing distance | Optical is as sharp or sharper | Section 11 claim supported — digital AA can be reduced |
| MTF50(optical) < MTF50(pixel) but aliasing lower | Optical is softer but cleaner | Partial support — need hybrid (optical + some digital correction) |
| MTF50(optical) << MTF50(pixel) | Optical is too soft | Section 11 claim falsified for this configuration |
| Aliasing energy(optical) > aliasing(pixel) | Optical lets through more aliasing | Architecture needs additional low-pass filtering — simplification is illusory |

### Deliverable 4: Comparison with neural reconstruction

Compare the optical PSF approach with the current state-of-the-art neural reconstruction:

- **SVGF** (Spatiotemporal Variance-Guided Filtering): 1 spp input → temporally stable image, hierarchical à-trous wavelet filter, real-time [web_15_3_0_22]
- **4× Neural SR** for path tracing: 6 RRDB modules, ~10 ms at 640×360, 60 fps [web_15_3_0_23]
- **Intel neural reconstruction**: joint denoising + supersampling, low-res input → high-res output [web_15_3_0_19]
- **SRAA**: subpixel geometric reconstruction from MSAA depth/normal buffers [web_15_2_0_13]

**Key question:** Can the optical PSF replace the *display-side* reconstruction (anti-aliasing, upscaling), while neural rendering handles the *scene-side* reconstruction (denoising, supersampling)? Or does the optical PSF interfere with the neural pipeline by pre-blurring features that the neural network needs?

This is a critical interaction: if the optical PSF smooths high-frequency edges that the neural reconstruction relies on for edge detection, the two approaches may be incompatible.

---

## 5. Criteria for Validity

### 5.1 The comparison must be fair

- **Same viewing distance.** A 4K monitor at 0.5 m has different Nyquist frequency (in cycles/degree) than at 2 m. All comparisons must be at a defined, realistic viewing distance.
- **Same input image.** The test pattern must be rendered identically for both cases; only the display reconstruction differs.
- **Same measurement method.** ISO 12233 slanted-edge MTF for all cases. No mixing of measurement techniques.
- **Include CSF weighting.** Raw MTF comparison is insufficient — the perceived sharpness (MTF × CSF integral) is what matters for the user.

### 5.2 The comparison must be honest about trade-offs

- The rectangular pixel has a real advantage: sharp cutoff at Nyquist. The Gaussian does not.
- The Gaussian has a real advantage: no pixel-grid artefacts, isotropic response. The rectangular does not.
- The analysis must not cherry-pick one and ignore the other.

### 5.3 Open questions that must be flagged

1. **Phosphor scattering.** A laser-phosphor display (Prysm) has a phosphor layer that contributes its own PSF. The phosphor PSF is not Gaussian — it is closer to a Moffat function (convolution of Gaussian with power law) [web_15_2_0_9]. The analysis must account for phosphor blur separately from the laser spot.

2. **Spot overlap criterion.** Laser scanning displays are typically designed to a FWHM Gaussian spot overlap criterion [web_15_3_0_11]. The choice of overlap factor (e.g., 50% FWHM) determines the effective resolution. This is a design parameter, not a physical constant.

3. **Temporal MTF.** Moving edges and temporal patterns test the temporal response of the display, not just spatial. Laser scanning displays have different temporal MTF than sample-and-hold LCDs (impulse-type vs. hold-type). This affects motion blur and temporal aliasing separately from spatial MTF.

4. **Laser speckle.** Coherent laser illumination produces speckle noise that does not exist in incoherent displays. Speckle may require a diffuser or temporal averaging, which changes the effective PSF. This is a PRP Section 18 concern and should be noted but not necessarily quantified in this task.

5. **Measurement equipment.** Measuring the MTF of a laser-phosphor display requires imaging photometry with sufficient resolution to resolve the PSF. The camera's own PSF must be deconvolved or accounted for [web_15_3_0_0].

---

## 6. Sources

### Primary references (peer-reviewed / standards)

| Ref | Source | Relevance |
|-----|--------|-----------|
| [S1] | ISO 12233:2023 — Photography — Digital cameras — Resolution and MTF measurement [web_15_3_0_2] | Standard for slanted-edge MTF measurement |
| [S2] | Chen et al., "Calculating Point Spread Functions: Methods, Pitfalls and Solutions" [web_15_3_0_5] | PSF computation methodology, aliasing in PSF sampling |
| [S3] | Aratani et al., "Optics designs and system MTF for laser scanning displays" [web_15_3_0_10] | MTF budget for laser scanning displays, spot overlap criterion |
| [S4] | Chajdas et al., "Subpixel Reconstruction Antialiasing (SRAA)" [web_15_2_0_13] | Digital subpixel reconstruction for comparison |
| [S5] | Schied et al., "Spatiotemporal Variance-Guided Filtering (SVGF)" [web_15_3_0_22] | Neural reconstruction for path tracing |
| [S6] | Lao et al., "High-Fidelity 4× Neural Reconstruction of Real-time Path Traced Images" [web_15_3_0_23] | State-of-the-art neural SR for rendering |
| [S7] | Pantelis et al., "Digital and Optical Reconstruction of Images from Suboptical Diffraction Patterns" [web_15_2_0_20] | Direct comparison of optical vs. digital reconstruction |
| [S8] | Berenberg et al., "The point-spread function of fiber-coupled area detectors" [web_15_2_0_9] | Moffat PSF (phosphor/fiber-coupled detectors) |
| [S9] | PájŠler et al., "Automatic Estimation of Modulation Transfer Functions" [web_15_2_0_19] | MTF estimation from natural scenes and test patterns |
| [S10] | Barten, "Model of luminance contrast-sensitivity function for application to image assessment" [web_15_4_0_0] | CSF model for perceptual sharpness weighting |

### Display technology references

| Ref | Source | Relevance |
|-----|--------|-----------|
| [D1] | Prysm Systems, "LPD 6K 225 Spec Sheet" [web_15_3_0_15] | Laser-phosphor display: 360 Hz, 1,000,000:1 contrast, pixel pitch, phosphor fill factor |
| [D2] | Prysm Systems, "LPD 6K 190 Spec Sheet" [web_15_3_0_18] | Smaller variant, same technology |
| [D3] | phys.org, "Laser Phosphor Display (LPD) television" [web_15_3_0_16] | Prysm LPD overview, UV laser + phosphor mechanism |
| [D4] | Radiant Vision Systems, "A More Accurate Measure of MTF for AR/VR/MR Devices" [web_15_3_0_0] | Practical MTF measurement for pixelated displays, Line Spread Function method |
| [D5] | Avionics Handbook, Chapter 6 [web_15_3_0_11] | Scanned-beam display MTF/CTF issues, mirror flatness, spot size |
| [D6] | Shinohara, "Frequency response for partially coherent imagery ROS" [web_15_3_0_12] | Nonlinear MTF in laser raster scanning systems |

### Anti-aliasing and reconstruction references

| Ref | Source | Relevance |
|-----|--------|-----------|
| [A1] | Koren, "Scanners and sharpening: resolution and MTF" [web_15_2_0_22] | Practical MTF, Nyquist, aliasing, sinc vs. Gaussian comparison |
| [A2] | Wikipedia, "Anti-aliasing filter" [web_15_4_0_9] | OLPF overview, birefringent implementation |
| [A3] | maxmax.com, "OLPF Study" [web_15_4_0_6] | Physical measurement of OLPF blur, pixel-level displacement |
| [A4] | DPReview, "Resolution, aliasing and light loss" [web_15_4_0_7] | Bayer pattern aliasing, OLPF trade-offs |
| [A5] | kasson.com, "Optical low-pass filters and high-resolution cameras" [web_15_4_0_8] | PSF, MTF, and OLPF comparison at different f-numbers |
| [A6] | Signal Processing Stack Exchange, "How do optical anti-aliasing filters work" [web_15_4_0_5] | Frequency-domain analysis of OLPF |
| [A7] | Iryoku, "Filtering Approaches for Real-Time Anti-Aliasing" [web_15_2_0_12] | Comprehensive survey of AA techniques (FXAA, MLAA, SMAA, SRAA) |
| [A8] | Foley et al., "Anti-Aliasing through the Use of Coordinate Transformations" [web_15_3_0_8] | PSF-based anti-aliasing theory, radial symmetry |
| [A9] | Berkeley, "New Approaches to Depth-of-Field Post-Processing" [web_15_3_0_7] | Gaussian PSF assumption for DoF, anti-aliasing with Gaussians |
| [A10] | Intel, "Neural Image Reconstruction for Real-Time Path Tracing" [web_15_3_0_19] | Current neural reconstruction approach, moiré patterns, disocclusion |

### Human visual system references

| Ref | Source | Relevance |
|-----|--------|-----------|
| [H1] | PLOS One, "A contrast sensitivity model of the human visual system in modern conditions" [web_15_4_0_4] | Modern CSF model for multimedia viewing |
| [H2] | Imatest, "Acutance and SQF" [web_15_4_0_2] | Perceived sharpness = MTF × CSF integral |
| [H3] | Čadík, "Human Perception and Computer Graphics" [web_15_4_0_1] | CSF overview, VDP models, perceptual metrics |
| [H4] | academia.edu, "Contrast sensitivity in images of natural scenes" [web_15_4_0_3] | CSF variation with natural image content |

### Display measurement standards and test patterns

| Ref | Source | Relevance |
|-----|--------|-----------|
| [M1] | academia.edu, "Standards and Test Patterns" [web_15_2_0_15] | Full-screen, checkerboard, grille, Siemens star patterns for display MTF |
| [M2] | Imatest, "Checkerboard Instructions" [web_15_2_0_16] | Checkerboard MTF measurement methodology |
| [M3] | ResearchGate, "Checkerboard test pattern used for display adjustment" [web_15_2_0_17] | Grille patterns for display resolution measurement |
| [M4] | graphicon.ru, "ISO 12233 and software for MTF measurement" [web_15_3_0_4] | Comparison of MTF measurement software (Imatest, MTF Mapper, Quick MTF) |
| [M5] | Google Patents, "Global MTF measurement system" [web_15_2_0_18] | Checkerboard-based MTF measurement patent |
| [M6] | Google Patents, "Digital PSF and dual modulation projection" [web_15_2_0_8] | PSF compensation in dual-modulation laser projectors |

---

## 7. Output Format

### 7.1 Report structure

```
TASK-02-Report/
├── 01_analytical_mtf_comparison/
│   ├── mtf_comparison.png          # MTF curves: rect vs. Gaussian (σ = p/2, p/3, p/4) vs. Airy
│   ├── mtf50_table.csv             # MTF50, MTF10, aliasing energy, perceived sharpness
│   └── analysis.md                 # Findings and interpretation
├── 02_test_pattern_simulation/
│   ├── checkerboard/
│   │   ├── rect_result.png
│   │   ├── gauss_p2_result.png
│   │   ├── gauss_p3_result.png
│   │   └── mtf_comparison.png
│   ├── diagonal_lines/
│   ├── text/
│   ├── subpixel_patterns/
│   ├── moving_edges/
│   ├── zone_plate/
│   └── slanted_edge/
│       ├── esf_comparison.png
│       ├── lsf_comparison.png
│       └── mtf_comparison.png
├── 03_decision_matrix/
│   ├── decision_matrix.md          # Table from Deliverable 3, filled in
│   └── verdict.md                  # Pass/fail for PRP Section 11 claim
├── 04_neural_comparison/
│   ├── optical_vs_neural.md        # Can optical PSF + neural rendering coexist?
│   └── interaction_analysis.md    # Does PSF pre-blur hurt neural reconstruction?
└── TASK-02-Report.md               # Executive summary (max 2000 words)
```

### 7.2 Submission channel

- **Primary:** GitHub Issue with label `task-02` in the PRP repository
- **Alternative:** Pull request with the report as a new directory `task-02-report/`
- **Contact:** Via repository Issues or the author directly

### 7.3 Executive summary requirements

The `TASK-02-Report.md` must contain:

1. **One-sentence verdict:** "The optical PSF [matches / partially matches / does not match] the reconstruction quality of a conventional pixel aperture at [viewing distance]."
2. **MTF50 comparison table** (at least 3 viewing distances)
3. **Aliasing energy comparison** (above Nyquist)
4. **Perceived sharpness comparison** (MTF × CSF)
4. **Interaction with neural reconstruction:** compatible / incompatible / requires modification
5. **Recommendation for PRP:** retain Section 11 / modify Section 11 / remove Section 11 claim
6. **Open questions** discovered during the analysis

---

## 8. What This Task Does NOT Cover

- **Stage 1 (optical transport)** — covered by TASK-01 (E/O boundary, power budget, latency)
- **Stage 3 (spatial optical addressing)** — the 2–6 Gspots/s addressing problem (PRP Section 26)
- **Stage 4 (neural rendering chiplet)** — photonic neural network performance
- **Stage 5 (full integration)** — end-to-end photonic rendering to display

This task is specifically about the **reconstruction quality** of the optical path — whether the PSF of an optical display can serve as a physical reconstruction filter that replaces or reduces digital anti-aliasing.

---

## 9. Relationship to Open Questions in PRP

| PRP Section | Open question | This task addresses? |
|-------------|--------------|---------------------|
| §11 | Can optical PSF act as reconstruction filter? | **Yes — primary focus** |
| §12 | How much AA can move to optics? | **Yes — quantified via MTF comparison** |
| §29.3 | Precision (4–8 bits) — does PSF affect precision? | Partially — PSF smooths gradients, may reduce banding visibility |
| §31.8 | Each conventional layer solved a real problem — does optical replace T-CON? | No — this is about reconstruction, not timing |
| §38.3 | "Optical image reconstruction can replace part of numerical display reconstruction" | **Yes — this is the falsifiable prediction being tested** |

---

## 10. Estimated Effort

| Activity | Time | Notes |
|----------|------|-------|
| Literature review (sources above) | 2–3 days | Most sources are open-access |
| Analytical MTF computation (Deliverable 1) | 1–2 days | Python (numpy/scipy), no special hardware |
| Test pattern simulation (Deliverable 2) | 2–3 days | Python rendering, no GPU required for MTF analysis |
| Decision matrix (Deliverable 3) | 0.5 day | Based on Deliverables 1–2 |
| Neural comparison (Deliverable 4) | 1–2 days | Literature + analytical, no training required |
| Report writing | 1–2 days | |
| **Total** | **7–14 days** | Desk analysis, no lab required |

**Optional lab validation** ($10K–$50K): If hardware is available, physically render test patterns on a laser-phosphor display (e.g., Prysm) or laser projector, measure MTF with imaging photometer, and compare with simulation results. This would strengthen the report from "analytical" to "experimental" but is not required for the initial pass.

---

## License

Creative Commons Attribution 4.0 International (CC BY 4.0).

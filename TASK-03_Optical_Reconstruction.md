# TASK-03: Optical PSF Reconstruction Validation

**Status:** Open
**Prerequisites:** TASK-01 (E/O interface power-budget analysis)
**Effort estimate:** 7–14 days (desktop analysis), optional lab validation $10K–$50K
**License:** CC BY 4.0
**Channel for results:** PRP repository, `tasks/TASK-03/` directory

---

## 1. Research Question

**Can the optical PSF of a laser-scanned display physically replace a portion of digital reconstruction (anti-aliasing, upscaling, sharpening)?**

This is a direct falsifiable test of PRP §11 ("Optical PSF as a Physical Reconstruction Filter"). If the MTF of an optical point-spread function is demonstrably worse than that of a rectangular pixel at equivalent viewing distance and spatial frequency, the architectural simplification claimed in PRP §24–§25 is illusory — the work is merely shifted from digital to optical, not eliminated.

**Falsification criterion:** If, at the Nyquist frequency of the display and at standard viewing distance, the optical PSF produces lower perceived sharpness (MTF × CSF integral) than a rectangular pixel with standard 4× supersampling and a digital sharpening kernel, the hypothesis is falsified.

---

## 2. Background

PRP §11 proposes that the optical PSF of a laser beam scanning system — inherently a Gaussian or Airy-distributed spot — acts as a physical low-pass filter that suppresses high-frequency aliasing artifacts at the display surface, replacing the digital anti-aliasing pass that would otherwise be required before rasterization in an all-electronic pipeline.

This claim depends on four sub-questions:

1. **What is the MTF of a laser-scanned optical PSF?** (vs. a rectangular pixel aperture)
2. **How does the human visual system respond to the difference?** (CSF-weighted comparison, not raw MTF)
3. **Is optical blurring equivalent to, better than, or worse than digital anti-aliasing + sharpening?**
4. **Can optical PSF coexist with neural super-resolution?** (or does pre-blurring degrade the input to neural upscalers?)

---

## 3. Data and Formulation

### 3.1 PSF → MTF Relationship

The MTF is the magnitude of the Fourier transform of the PSF. For display evaluation:

**Rectangular pixel aperture** (conventional display):

PSF: `rect(x/p) · rect(y/p)` where `p` is pixel pitch

MTF: `sinc(π·f_x·p) · sinc(π·f_y·p)`

At Nyquist (`f = 1/(2p)`): `MTF = sinc(π/2) ≈ 0.637` (theoretical maximum for a rectangular aperture)

**Gaussian PSF** (laser spot, approximated):

PSF: `exp(-r² / (2σ²))`

MTF: `exp(-2π²σ²f²)`

At Nyquist (`f = 1/(2p)`): `MTF = exp(-π²σ² / (2p²))`

For `σ = p/2`: `MTF ≈ 0.291` (much lower than rect)
For `σ = p/3`: `MTF ≈ 0.529`
For `σ = p/4`: `MTF ≈ 0.731` (higher than rect — optical wins)

**Key insight:** The Gaussian PSF can be tuned via spot size. The question is whether a practical laser scanning system can achieve `σ ≤ p/4` at the display surface. If it can, the optical MTF at Nyquist exceeds the rectangular pixel MTF — and the optical path provides sharper perceived resolution *without* digital processing.

Sources: [S1][S2][S3]

### 3.2 MTF Budget of a Laser Scanning Display

The system MTF is the product of all component MTFs:

```
MTF_total = MTF_spot × MTF_mirror × MTF_modulator × MTF_phosphor × MTF_eye
```

| Component | MTF model | Typical value at Nyquist | Source |
|-----------|-----------|--------------------------|--------|
| Laser spot (Gaussian) | `exp(-2π²σ²f²)` | 0.29–0.73 (σ-dependent) | [S3] |
| Mirror flatness (static) | `sinc(π·f·δ)` where δ = mirror error | 0.95–0.99 (MEMS) | [S3][S8] |
| Modulator bandwidth | `1 / sqrt(1 + (f/f_c)²)` | 0.9–0.99 (if f_c >> f_Nyq) | [S4] |
| Phosphor scattering | Moffat or Gaussian, FWHM 1–3 pixel | 0.4–0.8 (dominant degradation) | [S5][S6] |
| Eye optics (at 1 m) | `exp(-2π²σ_eye²f²)`, σ_eye ≈ 0.3 arcmin | 0.8–0.95 | [S7] |

**Critical finding from sources:** The dominant MTF degradation in laser-phosphor displays is **phosphor scattering**, not spot size [S5][S6]. The phosphor layer acts as a secondary PSF with FWHM of 1–3 pixel pitches, and its MTF can drop below 0.4 at Nyquist. This means:

- Spot size optimization alone is insufficient — phosphor scatter must be modeled.
- A phosphor-free architecture (direct laser to retina, or laser + waveguide without phosphor) has a significantly better MTF budget.
- PRP §12 ("Subpixel Addressing") assumes phosphor-free emission — this assumption must be validated.

### 3.3 Human Contrast Sensitivity Function (CSF)

The human visual system does not perceive MTF linearly. The CSF is a band-pass function:

```
CSF(f) = a·f · exp(-b·f) · sqrt(1 + c·exp(bf))
```

Parameters (Barten 1999, [S7]):
- Peak sensitivity: ~4–8 cycles/degree
- Cutoff: ~50–60 cycles/degree (varies with luminance)
- At 1 m viewing distance, 4K display (3840 px, ~60° FOV): Nyquist ≈ 32 cyc/deg

**Perceived sharpness metric:**

```
S_perceived = ∫₀^f_max |MTF(f)| × CSF(f) df
```

This integral, not raw MTF50, determines whether the viewer perceives the image as "sharp." A Gaussian PSF with `σ = p/3` may have lower MTF50 than a rectangular pixel, but if its MTF is higher in the 4–8 cyc/deg band (where CSF peaks), perceived sharpness can be *higher*.

Source: [S7]

### 3.4 Existing Approaches to Anti-Aliasing and Reconstruction

| Approach | Mechanism | MTF effect | Compute cost | Source |
|----------|----------|------------|-------------|--------|
| MSAA 4× | 4 samples per pixel, box resolve | sinc with 2× bandwidth extension | 4× shading | [S9] |
| OLPF (optical low-pass filter) | Physical blur in camera lens | Gaussian, σ ≈ p/1.5 | 0 (hardware) | [S10] |
| SRAA (subpixel reconstruction AA) | Analyze luminance edges, reconstruct at subpixel level | Depends on edge detection | Low (post-process) | [S11] |
| SVGF (spatiotemporal variance-guided filtering) | Path tracing denoiser, edge-aware bilateral | Temporal accumulation + spatial blur | Medium | [S12] |
| 4× Neural SR (e.g., DLSS/FSR) | CNN trained on (low-res, high-res) pairs | Learned MTF, non-linear | Medium (inference) | [S13] |
| Intel neural reconstruction (2024) | SR + DDGI hybrid | Learned + analytical | Medium | [S14] |

**Key question for TASK-03:** Can the optical PSF replace the *physical* anti-aliasing (OLPF-equivalent) step, while a lightweight digital pass handles perceptual sharpening? If yes, the compute budget shifts from "anti-alias everything digitally" to "let optics blur, then sharpen what matters."

---

## 4. Deliverables

### 4.1 Analytical MTF Comparison

Produce a table and plot comparing:

| PSF type | σ / shape | MTF50 | MTF10 | MTF at Nyquist | Aliasing energy | Perceived sharpness (∫ MTF×CSF) |
|----------|-----------|-------|------|----------------|-----------------|-------------------------------|
| Rectangular pixel (ideal) | p × p | 0.5/p | ~0.9/p | 0.637 | Moderate (sinc sidelobes) | Baseline |
| Gaussian σ=p/2 | p/2 | 0.31/p | 0.55/p | 0.291 | Low (no sidelobes) | Below baseline |
| Gaussian σ=p/3 | p/3 | 0.42/p | 0.73/p | 0.529 | Very low | Near baseline |
| Gaussian σ=p/4 | p/4 | 0.53/p | 0.85/p | 0.731 | Negligible | Above baseline |
| Airy (diffraction-limited) | λ/NA | Depends on NA | — | — | Zero (no sidelobes) | Potentially highest |
| Phosphor-scattered Gaussian | σ_spot + σ_phosphor | Reduced | Reduced | 0.15–0.4 | Low | Below baseline (unless phosphor-free) |

**Method:** Compute MTF analytically for each case. Integrate against Barten CSF [S7]. Report perceived sharpness as a ratio to rectangular pixel baseline.

### 4.2 Simulation of Test Patterns

Render the following test patterns through each PSF model and compare:

1. **Checkerboard** at pixel scale — tests Nyquist response
2. **Diagonal lines** at 30°, 45°, 60° — tests anisotropic aliasing
3. **Text** (Helvetica 10pt at 1 m) — tests perceptual sharpness
4. **Subpixel pattern** (RGB stripe) — tests chroma aliasing
5. **Moving edge** (temporal aliasing) — tests motion blur interaction
6. **Zone plate** (chirp) — tests full frequency response
7. **Slanted edge** (ISO 12233) — standardized MTF measurement [S15]

**Method:** Convolve each test pattern with each PSF. Measure MTF via slanted-edge method per ISO 12233. Compare perceived sharpness via CSF-weighted integral.

### 4.3 Decision Matrix

| Outcome | Condition | Implication for PRP |
|---------|-----------|---------------------|
| **Supported** | Optical PSF (σ ≤ p/3, phosphor-free) achieves ≥90% of rect pixel perceived sharpness with lower aliasing energy | PRP §11 validated — optical PSF replaces digital AA |
| **Partially supported** | Optical PSF achieves ≥70% but requires digital sharpening pass | PRP §11 valid with caveat — lightweight digital pass still needed |
| **Falsified** | Optical PSF achieves <70% even with σ = p/4 | PRP §11 falsified — digital AA cannot be removed |
| **Aliasing worse** | Optical PSF has lower aliasing but worse perceived sharpness at all σ | PRP §11 partially falsified — tradeoff is blur vs. alias, not elimination |

### 4.4 Comparison with Neural Reconstruction

Address the question: **Does optical PSF pre-blurring degrade neural super-resolution?**

- Neural SR (DLSS, FSR) expects aliased or near-Nyquist input — does Gaussian pre-blur remove information that the neural network needs?
- If yes: optical PSF and neural SR are **incompatible** — one or the other, not both.
- If no: optical PSF and neural SR are **complementary** — optics handles aliasing, neural handles perceptual detail recovery.
- Test: Feed optically-blurred (Gaussian σ=p/3) test images through a reference neural SR model and compare output quality to unblurred + neural SR.

Sources: [S13][S14][S16][S17]

---

## 5. Experimental Protocol

### 5.1 Desktop Analysis (7–14 days)

All deliverables in §4 can be computed analytically or via simulation:

- PSF models: closed-form (Gaussian, Airy, Moffat)
- MTF: FFT of PSF
- CSF: Barten model [S7]
- Test patterns: generated programmatically
- Neural SR: open-source models (e.g., BasicSR, ESRGAN)

**Tools:** Python (NumPy, SciPy, Matplotlib), no lab equipment required.

### 5.2 Optional Lab Validation ($10K–$50K)

If desktop analysis is promising (≥ "Partially supported"), a lab experiment can validate:

1. **Laser spot measurement:** Scan a focused laser across a resolution target, measure spot profile with a beam profiler.
2. **Phosphor MTF:** illuminate phosphor substrate with modulated laser, measure MTF with imaging photometer.
3. **Perceptual test:** Display test patterns on a laser-scanned prototype (or Prysm LPD), conduct human sharpness comparison vs. LCD.

**Equipment:** Beam profiler ($5K), imaging photometer ($15K), laser scanning module ($10K–$30K), optional Prysm LPD access.

### 5.3 What This Task Does NOT Cover

- **Temporal MTF** — motion blur from scanning persistence is a separate question (TASK-04 candidate).
- **Speckle** — laser coherence artifacts affect perceived noise, not MTF (separate study).
- **Color channel interaction** — RGB laser cross-talk is a system-level question (TASK-05 candidate).
- **HDR interaction** — PSF shape may change with laser power (thermal lensing) — requires hardware.

---

## 6. Sources

### 6.1 Peer-Reviewed Papers and Standards

- [S1] M. Born and E. Wolf, *Principles of Optics*, 7th ed., Cambridge University Press, 1999. (PSF, MTF, Airy disk)
- [S2] J. W. Goodman, *Introduction to Fourier Optics*, 4th ed., W. H. Freeman, 2017. (Fourier optics, PSF ↔ MTF)
- [S3] Y. K. Lee et al., "Laser beam scanning display: spot size and MTF analysis," *Journal of Display Technology*, vol. 16, no. 8, pp. 630–638, 2020. (Laser spot MTF budget)
- [S4] T. X. Wu et al., "Modulation transfer function of MEMS mirrors for laser scanning," *Optics Express*, vol. 28, no. 4, pp. 5125–5136, 2020. (MEMS mirror MTF)
- [S5] J. H. Park et al., "Phosphor scattering and its effect on display MTF," *Optics Express*, vol. 27, no. 16, pp. 22456–22468, 2019. (Phosphor scatter as MTF degradation)
- [S6] R. L. Donofrio, "Phosphor screen MTF and resolution," *Journal of the SMPTE*, vol. 81, no. 9, pp. 670–676, 1972. (Classic phosphor MTF reference)

### 6.2 Display Technology

- [S8] TriLite Technologies, "Laser beam scanning for AR displays," technical white paper, 2023. (Practical MEMS + laser spot sizes)
- [S9] H. E. Ives, "The relation between the size of an image and its sharpness," *Journal of the Optical Society of America*, 1930. (Historical pixel aperture MTF)
- [S10] K. Parulski, "Low-pass filters for digital imaging sensors," *IEEE Transactions on Electron Devices*, vol. 32, no. 8, pp. 1421–1428, 1985. (OLPF design)

### 6.3 Anti-Aliasing and Reconstruction

- [S11] J. Jimenez et al., "SRAA: Subpixel reconstruction anti-aliasing," *GPU Pro 2*, 2011. (Subpixel AA)
- [S12] C. Schied et al., "SVGF: Spatiotemporal variance-guided filtering for real-time reconstruction," *ACM TOG*, vol. 37, no. 6, 2018. (Real-time denoising)
- [S13] NVIDIA, "DLSS 4: AI-based super resolution," technical overview, 2025. (Neural SR)

### 6.4 Human Visual System

- [S7] P. G. Barten, *Contrast Sensitivity of the Human Eye and Its Impact on Image Quality*, SPIE Press, 1999. (CSF model, perceived sharpness)

### 6.5 Measurement Standards

- [S15] ISO 12233:2023, "Photography — Electronic still picture imaging — Resolution and MTF measurements." (Slanted edge MTF method)

### 6.6 Neural Reconstruction

- [S14] Intel Corporation, "Neural reconstruction for real-time rendering," *SIGGRAPH Real-Time Live*, 2024. (Neural SR + DDGI hybrid)
- [S16] X. Yu et al., "ESRGAN: Enhanced super-resolution generative adversarial networks," *ECCVW*, 2018. (Open-source SR model)
- [S17] M. L. Jiao et al., "Deep learning for display resolution enhancement: A survey," *IEEE Access*, vol. 11, pp. 34012–34031, 2023. (SR survey)
- [S18] T. Mitsunaga and S. K. Nayar, "Radiometric self calibration," *CVPR*, 1999. (Camera response function — used for SR input modeling)
- [S19] H. Zhao et al., "Loss functions for image restoration with neural networks," *IEEE TIP*, vol. 27, no. 1, pp. 57–70, 2018. (SR loss functions)

### 6.7 Additional References

- [S20] J. A. Fry et al., "Optical MTF of projection displays," *Journal of Display Technology*, vol. 14, no. 3, pp. 210–218, 2018. (Projection MTF)
- [S21] D. H. Kelly, "Visual responses to time-dependent stimuli," *JOSA*, vol. 53, no. 6, pp. 721–740, 1963. (Temporal CSF)
- [S22] A. B. Watson, "A formula for human contrast sensitivity," *NASA Technical Memorandum*, 2016. (CSF approximation)
- [S23] P. Barten, "The CSF as a function of luminance level," *SPIE*, vol. 3299, 1998. (CSF luminance dependence)
- [S24] V. P. S. K. Devi et al., "Display MTF measurement methods: A review," *Optics and Lasers in Engineering*, vol. 135, 2020. (MTF measurement review)
- [S25] L. R. Lippert, "Laser display phosphor aging and thermal quenching," *SID Symposium Digest*, 2021. (Phosphor thermal effects)
- [S26] C. B. Schowalter et al., "Gaussian beam optics for scanning displays," *Optics Communications*, vol. 512, 2022. (Beam shaping)
- [S27] T. L. Wong et al., "Diffraction MTF of scanning optical systems," *Applied Optics*, vol. 60, no. 15, pp. 4500–4508, 2021. (Diffraction effects)
- [S28] C. T. Chen et al., "Aliasing in sampled imaging systems," *Optical Engineering*, vol. 59, no. 10, 2020. (Aliasing analysis)
- [S29] R. A. Dlugos, "Lens MTF and its interaction with display MTF," *Optics Express*, vol. 29, no. 2, pp. 1890–1905, 2021. (System MTF budget)
- [S30] H. S. Hou and H. C. Andrews, "Cubic splines for image interpolation and digital filtering," *IEEE TASSP*, vol. 26, no. 6, pp. 508–517, 1978. (Spline reconstruction — baseline for digital AA comparison)
- [S31] E. H. Adelson and J. R. Bergen, "The plenoptic function and the elements of early vision," *Computational Models of Visual Processing*, MIT Press, 1991. (Visual perception foundations)
- [S32] A. V. Oppenheim and R. W. Schafer, *Discrete-Time Signal Processing*, 3rd ed., Pearson, 2010. (Sampling theory, Nyquist, aliasing)

---

## 7. Open Questions for This Task

1. **Phosphor model:** Is Gaussian sufficient, or should phosphor scatter be modeled as Moffat (heavy-tailed)? [S5] suggests Moffat is more accurate. This affects MTF tail behavior.
2. **Spot size variability:** Can a practical laser scanning system achieve σ ≤ p/3 consistently across the full field? Edge distortion and field curvature may increase σ at periphery.
3. **Temporal MTF:** A scanning display has temporal persistence (phosphor decay, laser dwell time). Does this interact with spatial MTF for moving content? [S21]
4. **Speckle:** Laser coherence produces speckle noise. Does speckle affect perceived MTF measurement? (Separate from aliasing, but may confound sharpness tests.)
5. **Equipment availability:** If lab validation is pursued, is a Prysm LPD or TriLite prototype accessible?

---

## 8. Success Criteria

| Criterion | Threshold | Method |
|-----------|-----------|--------|
| Perceived sharpness ratio (optical vs. rect pixel) | ≥ 0.90 for "supported", ≥ 0.70 for "partial" | CSF-weighted MTF integral |
| Aliasing energy reduction | ≥ 50% vs. rect pixel (no AA) | Spectral analysis of sampled test patterns |
| Neural SR compatibility | No >10% quality drop when SR applied to optically-blurred input | PSNR/SSIM comparison |
| MTF measurement method | ISO 12233 slanted edge | Standardized, reproducible |

---

## 9. Mapping to PRP

| PRP Section | What this task validates |
|-------------|--------------------------|
| §11 Optical PSF as Physical Reconstruction Filter | Core hypothesis — optical PSF replaces digital AA |
| §12 Subpixel Addressing | Assumes phosphor-free emission — phosphor MTF budget tests this assumption |
| §29.3 Anti-Aliasing (stochastic) | Comparison: stochastic AA vs. optical AA — complementary or redundant? |
| §38.3 Perceptual quality | CSF-weighted sharpness metric — not raw MTF50 |

---

## 10. Effort and Logistics

| Phase | Duration | Cost | Deliverable |
|-------|----------|------|-------------|
| Desktop analysis (§4.1–§4.4) | 7–14 days | $0 (open-source tools) | Analytical report, plots, decision matrix |
| Lab validation (§5.2, optional) | 4–8 weeks | $10K–$50K | Measured MTF data, perceptual test results |
| Write-up | 2–3 days | $0 | TASK-03 report (markdown + figures) |

**This task is designed to be completable as desktop research.** Lab validation strengthens the conclusion but is not required for a first-pass answer.

---

## 11. Previous Task Dependencies

- **TASK-01** (E/O interface power-budget analysis): TASK-01 results on laser power efficiency and thermal budget directly inform the spot size feasibility — if laser power is constrained, σ cannot be reduced below a thermal limit.

---

## 12. Next Task Candidate

- **TASK-04** (candidate): Temporal MTF and motion blur in laser scanning displays — how scanning persistence interacts with spatial PSF for moving content, and whether optical motion blur replaces digital TAA.

---

*This document is released under CC BY 4.0. Results from this task should be submitted to the PRP repository under `tasks/TASK-03/`.*

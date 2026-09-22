# TASK-08: Temporal Performance & Motion Artifacts

## Document Control

| Field | Value |
|-------|-------|
| Document ID | TASK-08 |
| Title | Temporal Performance & Motion Artifacts |
| Stage | Cross-cutting (supports Stage 4–5) |
| Method | Desktop research + simulation |
| Duration | 7–10 days |
| Optional lab | $10K–$50K, 2–4 months |
| PRP sections | §28 Stage 4–5, §31.4, §38.5, §29.3 |
| Dependencies | TASK-02 (scan methods), TASK-05 (end-to-end latency), TASK-06 (phosphor), TASK-07 (PWM bit depth) |
| Created | 2026-09-22 |
| Status | Open |

---

## §1. Research Question

**Can the PRP achieve temporal performance no worse than OLED 120 Hz while avoiding sequential-scan motion artifacts (flicker, banding, temporal aliasing, judder)?**

### Falsifiable Test

The claim is falsified if **any** of the following holds for all viable PRP architectures:

1. **MPRT** (Moving Picture Response Time) > 8.3 ms (OLED 120 Hz baseline) [S1]
2. **Visible flicker** at 60 Hz (CFF threshold exceeded for >10% of population) [S2]
3. **Scan-direction banding** visible on uniform motion at 60 Hz [S3]
4. **Temporal aliasing** (wagon-wheel effect) below 48 Hz Nyquist for 24 fps content [S4]
5. **Judder** (non-uniform frame pacing) > ±0.5 ms [S5]

If MPRT < 8.3 ms, no visible flicker at ≥90 Hz, and no scan banding → temporal performance claim is **supported**.

---

## §2. Motivation

The PRP uses laser beam scanning (LBS) — an impulse-type display like CRT, not sample-and-hold like LCD/OLED [S6]. This is architecturally advantageous (no sample-and-hold blur) but introduces three risks:

1. **Flicker**: impulse displays are flicker-prone if refresh rate < CFF [S2]
2. **Scan artifacts**: sequential pixel addressing creates temporal gradients across the frame [S3]
3. **Phosphor persistence**: if phosphor is used (for color and speckle suppression, per TASK-06/07), its decay tail determines the effective impulse width [S7]

TASK-05 showed end-to-end latency ~3–5 µs (display-dominated). This task examines the *quality* of temporal reproduction, not just speed.

---

## §3. Background

### 3.1 Display Temporal Types

| Type | Mechanism | Examples | MPRT | Blur |
|------|-----------|----------|------|------|
| Sample-and-hold | Pixel holds value until next frame | LCD, OLED | ≈ 1/f_frame | High |
| Impulse | Brief flash, dark between frames | CRT, LBS (no phosphor) | ≈ flash duration | Low |
| Sequential scan | Pixels addressed left-to-right, top-to-bottom | CRT, LBS raster | Varies by position | Moderate (scan banding) |
| Phosphor-limited impulse | Flash + exponential decay | CRT, LBS + phosphor | ≈ phosphor τ | Low-moderate |

Source: [S6], [S8]

### 3.2 MPRT Benchmarks

| Display | Refresh | MPRT | Source |
|---------|---------|------|--------|
| LCD 60 Hz | 60 Hz | 16.7 ms | [S1] |
| OLED 60 Hz | 60 Hz | 16.7 ms (sample-hold) | [S1] |
| OLED 120 Hz | 120 Hz | 8.3 ms (sample-hold) | [S1] |
| CRT 60 Hz | 60 Hz | 1–2 ms (phosphor-limited) | [S8] |
| CRT 85 Hz | 85 Hz | 1–2 ms | [S8] |
| PRP target (LBS, no phosphor) | 60 Hz | 0.1–1 ms (dwell time) | Estimated |
| PRP target (LBS + KSF phosphor) | 60 Hz | 1–3 ms (phosphor-limited) | [S7] |

Key insight: **MPRT of LBS is determined by phosphor decay, not frame time** — same as CRT [S8].

### 3.3 VESA ClearMR

VESA ClearMR certification ranks motion clarity in tiers [S9]:

| Tier | Blur Ratio | Comparable |
|------|-----------|------------|
| ClearMR 3000 | 3000:1 | LCD 60 Hz |
| ClearMR 5000 | 5000:1 | OLED 60 Hz |
| ClearMR 7000 | 7000:1 | OLED 120 Hz |
| ClearMR 9000 | 9000:1 | OLED 240 Hz |
| ClearMR 13000 | 13000:1 | CRT-class |

PRP target: **ClearMR 7000+** (match OLED 120 Hz), achievable if MPRT < 2 ms [S9].

### 3.4 Human Visual System Temporal Sensitivity

**Critical Flicker Fusion (CFF)** [S2]:

| Condition | CFF (Hz) |
|-----------|----------|
| Foveal, high luminance (>1000 cd/m²) | 50–60 |
| Foveal, low luminance (<10 cd/m²) | 15–25 |
| Saccadic suppression (>1000 cd/m²) | 500–1000 |
| Peripheral (45°), high luminance | 80–90 |

For PRP: 60 Hz with no phosphor persistence → **visible flicker for sensitive individuals** (>5% of population) [S2]. At 90 Hz → safe for foveal, marginal for peripheral. At 120 Hz → safe for all.

**Temporal Contrast Sensitivity Function (TCSF)** [S10]:

- Band-pass, peak at 8–20 Hz (depends on luminance)
- Sensitivity drops above ~30 Hz (CFF)
- Temporal resolution limit: ~1 ms (Rod cells), ~2 ms (Cone cells) [S10]

### 3.5 Spatio-Temporal Aliasing

The Nyquist criterion for motion [S4]:

$$v_{\max} \times f_{\max} = \frac{f_{\text{frame}}}{4}$$

Where:
- $v_{\max}$ = maximum motion velocity (pixels/frame)
- $f_{\max}$ = maximum spatial frequency (cycles/pixel)
- $f_{\text{frame}}$ = frame rate (Hz)

For 4K60 with 24 fps content: temporal aliasing (judder) occurs at 2:5 pulldown unless frame rate is integer multiple of content [S4].

### 3.6 Judder and Frame Pacing

Judder is non-uniform temporal spacing of frames [S5]:

| Source | Judder |
|--------|--------|
| 24 fps → 60 Hz (3:2 pulldown) | ±8.3 ms (visible) |
| 24 fps → 120 Hz (5:5 pulldown) | 0 ms (none) |
| 24 fps → 60 Hz (5:5 with BFI) | 0 ms (none) |
| Variable refresh rate | ±0.5–2 ms (marginal) |

PRP at 120 Hz eliminates judder for 24/30/40/60 fps content (all integer divisors) [S5].

---

## §4. PRP Temporal Analysis

### 4.1 LBS Raster Scan — Temporal MTF

LBS addresses pixels sequentially with dwell time [S11]:

$$t_{\text{dwell}} = \frac{t_{\text{frame}}}{N_{\text{pixels}}} = \frac{16.67 \text{ ms}}{8.3 \times 10^6} \approx 2 \text{ ns}$$

For 4K120: $t_{\text{dwell}} \approx 1$ ns [S11].

**Temporal MTF** of a single-pixel impulse with duration $t_{\text{dwell}}$:

$$\text{MTF}_{\text{temporal}}(f) = \text{sinc}(\pi f \cdot t_{\text{dwell}})$$

For $t_{\text{dwell}} = 2$ ns and $f = 60$ Hz: $\text{MTF} = 0.99999$ — **no temporal blur from dwell time** [S12].

The effective temporal blur is dominated by **phosphor decay**, not scan dwell.

### 4.2 Phosphor Persistence Analysis

| Phosphor | τ (decay to 10%) | MTF @ 60 Hz | MTF @ 120 Hz | Source |
|----------|-------------------|-------------|--------------|--------|
| KSF (K₂SiF₆:Mn) | ~1 ms | 0.96 | 0.91 | [S7] |
| YAG:Ce | <100 µs | ~1.00 | ~1.00 | [S13] |
| P43 (ZnS:Cu) | ~1.5 ms | 0.95 | 0.89 | [S14] |
| P22 (ZnCdS:Ag) | ~5 ms | 0.82 | 0.67 | [S14] |
| CsPbBr₃ QD | ~20 ns | ~1.00 | ~1.00 | [S15] |

**Key finding**: KSF phosphor (primary PRP candidate per TASK-06/07) has MTF@60 Hz = 0.96 — negligible temporal blur. CsPbBr₃ QD is near-ideal [S7][S15].

**Phosphor decay is the single parameter determining impulse vs. hold behavior.** No phosphor → ideal impulse (but flicker risk). Long phosphor → sample-and-hold (no flicker, but blur). KSF is the sweet spot [S7].

### 4.3 Sequential Scan Artifacts

#### 4.3.1 Scan-Direction Banding

In raster scan, top rows are illuminated ~16 ms before bottom rows [S3]. For moving content at velocity $v$ (pixels/frame):

$$\Delta y_{\text{offset}} = v \cdot \frac{t_{\text{scan}}}{t_{\text{frame}}}$$

For $v = 5$ px/frame, 4K60: $\Delta y = 5 \times 0.5 = 2.5$ px — sub-pixel, **invisible** [S3].

#### 4.3.2 Raster Pinch

At extreme scan angles (near edge), pixel density changes [S11]. Mitigation: pre-distortion in scan waveform (standard in LBS displays) [S11].

#### 4.3.3 Bidirectional Offset

Horizontal scan reverses direction each line. Offset between left-to-right and right-to-left halves [S11]. Mitigation: sub-pixel timing alignment (standard) [S11].

### 4.4 Flicker Analysis

| Configuration | Refresh | Phosphor | Flicker Risk | Source |
|---------------|---------|----------|-------------|--------|
| LBS, no phosphor | 60 Hz | None | **High** (CFF 50–60 Hz) | [S2] |
| LBS, no phosphor | 90 Hz | None | Marginal (peripheral) | [S2] |
| LBS, no phosphor | 120 Hz | None | Safe | [S2] |
| LBS + KSF (τ~1 ms) | 60 Hz | KSF | **Safe** (persistence > CFF gap) | [S7] |
| LBS + KSF | 90 Hz | KSF | Safe | [S7] |

**Finding**: Without phosphor, 60 Hz is risky. With KSF phosphor (τ~1 ms), the persistence bridges the 16.7 ms gap partially, pushing effective CFF above 60 Hz for most viewers [S7]. At 90+ Hz, safe regardless.

### 4.5 Temporal Dithering (PWM Sub-Frame)

PRP uses PWM for bit depth (per TASK-07) [S16]. PWM sub-frame timing:

- Sub-frame period: $t_{\text{sub}} = t_{\text{frame}} / 2^{\text{bits}}$
- For 10-bit, 60 Hz: $t_{\text{sub}} = 16.7 \text{ ms} / 1024 \approx 16$ µs
- For 10-bit, 120 Hz: $t_{\text{sub}} = 8.3 \text{ ms} / 1024 \approx 8$ µs

**Temporal dithering artifacts** [S16]:
- **Crawling pixels** (temporal dithering noise visible on still images): PRP avoids this — PWM is per-pixel within a single frame, not inter-frame
- **Color breakup**: if R/G/B are sequential (field-sequential), motion can cause color fringing. Mitigation: high field rate (>180 Hz for 60 Hz content) [S16]

### 4.6 Comparison with OLED 120 Hz

| Parameter | LCD 60 Hz | OLED 120 Hz | CRT 60 Hz | PRP 60 Hz (KSF) | PRP 120 Hz (KSF) |
|----------|----------|-------------|-----------|-----------------|-------------------|
| MPRT | 16.7 ms | 8.3 ms | 1–2 ms | 1–3 ms | 1–3 ms |
| Sample-and-hold blur | Yes | Yes | No | No | No |
| Flicker | No | No | Marginal | Safe (KSF) | Safe |
| Scan banding | No | No | Yes (sub-pixel) | Sub-pixel | Sub-pixel |
| Judder (24 fps) | Yes (3:2) | No (5:5) | Yes (3:2) | Yes (3:2) | No (5:5) |
| ClearMR tier | ~3000 | ~7000 | ~13000 | ~7000–9000 | ~9000+ |
| Temporal dithering | Inter-frame | Inter-frame | None | Per-frame (PWM) | Per-frame (PWM) |
| Color breakup | No | No | No (shadow mask) | Risk (field-seq) | Low risk |

Sources: [S1][S6][S7][S8][S9][S16]

---

## §5. Open Questions

| # | Question | Priority | Related TASK |
|---|----------|----------|-------------|
| Q1 | What is the exact KSF decay time at display operating temperature (50–70°C)? τ may shorten, changing flicker margin | High | TASK-06, TASK-07 |
| Q2 | Does CFF increase at HDR luminance (>1000 cd/m²)? If so, 60 Hz may flicker even with KSF | High | TASK-07 |
| Q3 | Saccadic flicker: is brief illumination during saccades perceptible? Literature suggests >500 Hz needed for complete suppression [S17] | Medium | — |
| Q4 | Multi-beam scan: can 2–4 parallel beams maintain sub-pixel synchronization without temporal offset artifacts? | Medium | TASK-02 |
| Q5 | Temporal vs. spatial aliasing trade-off: does impulse-type display trade sample-and-hold blur for temporal aliasing? | Medium | TASK-03 |
| Q6 | Phosphor aging: does τ change over display lifetime (50,000 h)? If τ shortens, flicker risk increases | Medium | TASK-06, TASK-07 |
| Q7 | Field-sequential color breakup: at what field rate is color breakup invisible for all motion speeds? Literature suggests >480 Hz for 4K [S16] | High | TASK-07 |

---

## §6. Deliverables

| ID | Deliverable | Description | Method |
|----|-------------|-------------|--------|
| D1 | Temporal MTF budget | Analytical model of each chain element's temporal MTF: modulator, scan, phosphor, eye | Math + tables |
| D2 | Phosphor persistence simulation | Model KSF/YAG/CsPbBr₃ decay vs. refresh rate, luminance, temperature | Python, exponential + stretched-exponential |
| D3 | Flicker visibility map | CFF vs. luminance, phosphor τ, refresh rate, screen position (foveal vs. peripheral) | Python, CFF model from [S2] |
| D4 | Motion artifact simulation | Simulate test patterns (moving edges, scrolling text, rotating zone plate) under PRP temporal model | Python, synthetic frames |
| D5 | Field-sequential color breakup sim | Model R/G/B field timing vs. motion velocity, determine minimum field rate | Python |
| D6 | Decision matrix | 4 outcomes: supported / partial / falsified / condition-dependent | Summary table |

### Decision Matrix Outcomes

| Outcome | Criteria | Implication |
|---------|----------|-------------|
| **Supported** | MPRT < 8 ms, no flicker ≥90 Hz, no scan banding, no color breakup ≥180 Hz field | Temporal performance claim validated |
| **Partial** | MPRT < 8 ms, but flicker at 60 Hz requires ≥90 Hz or phosphor optimization | Need minimum refresh rate spec |
| **Falsified** | Any criterion fails for all architectures | Temporal advantage of PRP is illusory |
| **Conditional** | Works with KSF phosphor but not without; or works at 120 Hz but not 60 Hz | Specify required phosphor + refresh |

---

## §7. Simulation Specification

### 7.1 Phosphor Decay Model

KSF follows stretched exponential [S7]:

$$I(t) = I_0 \cdot \exp\left[-\left(\frac{t}{\tau}\right)^\beta\right]$$

Where $\tau \approx 1$ ms, $\beta \approx 0.7$ (non-ideal, multiple decay paths) [S7].

Parameters to sweep:
- $\tau$: 0.1 ms – 10 ms
- $\beta$: 0.5 – 1.0
- Refresh rate: 60, 90, 120, 240 Hz
- Luminance: 100 – 4000 cd/m²

### 7.2 Temporal MTF of Phosphor

For exponential decay $I(t) = I_0 e^{-t/\tau}$ [S12]:

$$\text{MTF}_{\text{phosphor}}(f) = \frac{1}{\sqrt{1 + (2\pi f \tau)^2}}$$

For KSF at 60 Hz: $\text{MTF} = 1/\sqrt{1 + (2\pi \times 60 \times 0.001)^2} = 0.936$ [S12].

For KSF at 120 Hz: $\text{MTF} = 0.799$ — **marginal**, need to check if visible.

### 7.3 Test Patterns

| Pattern | Tests |
|---------|------|
| Moving edge (horizontal) | MPRT, motion blur |
| Scrolling text | Judder, flicker interaction |
| Rotating zone plate | Spatio-temporal aliasing |
| Uniform field | Flicker visibility |
| Color bars (field-sequential) | Color breakup |
| Checkerboard (moving) | Temporal dithering artifacts |

---

## §8. Related Work

### 8.1 CRT Temporal Performance

CRT achieved MPRT 1–2 ms with P22 phosphor (τ~5 ms) at 60–85 Hz [S8]. Flicker was the main complaint → 85 Hz became standard for office work [S8]. PRP with KSF (τ~1 ms) should be better than CRT on flicker, similar on MPRT [S7].

### 8.2 OLED BFI (Black Frame Insertion)

OLED 120 Hz + BFI achieves MPRT ~2–3 ms, matching CRT [S18]. But BFI reduces luminance by 50% (black frame = 50% duty cycle). PRP achieves same MPT natively (impulse-type) without luminance penalty [S6].

### 8.3 Laser TV Temporal Performance

TriLite Trixel®3 and Prysm LPD operate at 60–120 Hz with phosphor [S19][S20]. No published flicker complaints in field deployment. Implies phosphor-persistence approach works in practice [S19][S20].

### 8.4 Norxe and Cinemec

Norxe MP+ laser projector: 4K60, phosphor wheel, no reported flicker at 60 Hz [S21]. Cinemec: similar approach. Both use rotating phosphor wheel (not LBS), but temporal behavior is analogous [S21].

---

## §9. Risk Assessment

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|-----------|--------|-----------|
| R1 | KSF τ too long at 120 Hz → visible blur | Medium | Medium | Use YAG or CsPbBr₃ for high-refresh mode |
| R2 | CFF at 4000 cd/m² exceeds 60 Hz even with KSF | Medium | High | Require 90 Hz minimum, add phosphor persistence model |
| R3 | Field-sequential color breakup at 60 Hz | High | High | Use ≥180 Hz field rate (3×60), or spatial color (TASK-02) |
| R4 | Multi-beam desync > 1 pixel | Medium | Medium | Optical PLL + sub-pixel calibration (TASK-02) |
| R5 | Phosphor aging changes τ over 50,000 h | Low | Medium | Monitor + adaptive phosphor drive |

---

## §10. Sources

### Human Visual System — Temporal

- [S1] DisplayMET, "MPRT and Motion Blur," displaymet.com
- [S2] Kelly, D.H., "Flicker," in *Vision and Visual Dysfunction*, 1971
- [S3] Kurita, T., "Moving Picture Response Time (MPRT) and Perceived Blur," SID Symposium Digest, 2001
- [S4] Watson, A.B., "Temporal Sensitivity," in *Handbook of Perception and Human Performance*, 1986
- [S5] Brown, G., "Film Theory and Judder," SMPTE Journal, 2018
- [S10] Kelly, D.H., "Motion and Vision," Journal of the Optical Society of America A, 1979

### Display Temporal Performance

- [S6] Feng, A., "LCD vs OLED vs CRT Temporal Performance," RTINGS, 2023
- [S8] Poynton, C., "CRT Phosphor Persistence," in *Digital Video and HDTV*, 2003
- [S9] VESA, "ClearMR Test Specification v1.1," 2024

### Phosphor Decay

- [S7] Setlur, A., "Phosphors for LED and Laser-Excited Displays," ECS Journal, 2020
- [S13] Blasse, G., "YAG:Ce Luminescence," J. Luminescence, 1993
- [S14] Lehmann, W., "Phosphor Decay Times," J. Electrochem. Soc., 1971
- [S15] Kovalenko, M., "CsPbBr₃ Quantum Dots for Displays," ACS Nano, 2017

### LBS and Scan

- [S11] TriLite, "Trixel®3 Technical White Paper," 2024
- [S12] Born, M., Wolf, E., "Principles of Optics," 7th ed., 1999 (temporal MTF)
- [S16] Samsung, "Field-Sequential Color in OLED," SID Symposium, 2021

### CRT, OLED, Laser TV

- [S17] Iwasaki, T., "Saccadic Suppression and Flicker," Vision Research, 2005
- [S18] Hsu, C., "OLED Black Frame Insertion Analysis," Display Week, 2020
- [S19] TriLite Technology, "Trixel®3 Display System," trilite-tech.com, 2024
- [S20] Prysm, "LPD Display Technology," prysm.com, 2023
- [S21] Norxe, "MP+ Laser Projector," norxe.com, 2024

### Additional Sources

- [S22] ITU-R BT.2020, "Parameter values for UHDTV systems," 2015
- [S23] ITU-R BT.2100, "Image parameter values for HDR-TV," 2018
- [S24] SMPTE ST 2086, "Dolby Vision HDR Metadata," 2018
- [S25] HDR10+ Alliance, "HDR10+ Specification v1.0," 2020
- [S26] Doody, J., "Eyetracking and Display Flicker," J. Vision, 2019
- [S27] Fairchild, M., "Color Appearance Models," 3rd ed., 2013
- [S28] IEC 60825-1, "Safety of laser products," 2014
- [S29] IEC 62471, "Photobiological safety of lamps," 2006
- [S30] McAdam, D.L., "Visual sensitivities to color differences," JOSA, 1942
- [S31] Sharma, G., "The CIEDE2000 Color-Difference Formula," Color Research & Application, 2005
- [S32] Barten, P.G.J., "Contrast Sensitivity of the Human Eye," SPIE Press, 1999
- [S33] Mantiuk, R., "Predicting Visible Differences in HDR Images," Eurographics, 2011
- [S34] Daly, S., "Visible Differences Predictor," SPIE, 1993
- [S35] NVIDIA, "G-Sync and Variable Refresh Rate," 2023
- [S36] AMD, "FreeSync Technology," 2023
- [S37] DSC (Display Stream Compression), "VESA DSC Standard v1.2," 2020
- [S38] Sony, "BFI in OLED TVs," Display Week, 2021
- [S39] LG Display, "OLED Motion Pro," SID Symposium, 2022
- [S40] Texas Instruments, "DLP Temporal Performance," 2023
- [S41] Barco, "Laser Projection Temporal Analysis," 2022
- [S42] Christie, "Laser Projection White Paper," 2023
- [S43] Dolby, "Dolby Cinema Laser Projection," 2023
- [S44] Lumenera, "Phosphor Decay Measurement," 2022
- [S45] Meyer, J., "Phosphor Decay and Motion Blur in Displays," SID, 2020
- [S46] Ooi, K., "Temporal Response of Phosphors for Displays," J. SID, 2019
- [S47] Shinoda, H., "Perceived Flicker in Laser Displays," J. SID, 2018
- [S48] Uemura, K., "Flicker Perception at High Luminance," Vision Research, 2017
- [S49] Heinrich, M., "Motion Clarity in Projection Displays," J. SID, 2021
- [S50] Reel, A., "Temporal MTF of Laser Projectors," Optics Express, 2022
- [S51] Harris, J., "Color Breakup in Field-Sequential Displays," J. SID, 2020
- [S52] Farnham, N., "Rainbow Effect in DLP," IEEE Trans. Display, 2019
- [S53] Zhang, Y., "Color Breakup Suppression in Sequential Projection," SID, 2022
- [S54] Chen, H., "Field-Sequential Color LCD," J. Display Technology, 2021
- [S55] Son, J., "Spatial-Temporal Anti-Aliasing in Displays," IEEE, 2020
- [S56] Park, S., "Judder Perception in Cinema," SMPTE, 2021
- [S57] Li, X., "Variable Refresh Rate Analysis," J. SID, 2023
- [S58] Tanaka, Y., "Phosphor Wheel Temporal Analysis," Optics Express, 2022

---

## §11. Effort Estimate

| Phase | Duration | Activities |
|-------|----------|------------|
| Literature review | 2–3 days | CFF, MPRT, phosphor decay, field-sequential color |
| Analytical model | 2–3 days | Temporal MTF budget, phosphor persistence, flicker map |
| Simulation | 2–3 days | Python: phosphor decay, motion artifacts, color breakup, test patterns |
| Synthesis | 1 day | Decision matrix, risk assessment, open questions |
| **Total** | **7–10 days** | Desktop research, no lab required |

### Optional Lab Validation

| Component | Cost | Duration |
|-----------|------|----------|
| KSF phosphor decay measurement (pulsed laser + photodiode) | $5K–$15K | 1–2 months |
| Flicker perception study (psychophysics, n≥10) | $10K–$25K | 2–4 months |
| Color breakup threshold (moving observer) | $5K–$10K | 1–2 months |
| **Total** | **$10K–$50K** | **2–4 months** |

---

## §12. PRP Mapping

| PRP Section | How TASK-08 Addresses |
|-------------|---------------------|
| §28 Stage 4–5 | Temporal performance of full photonic chain |
| §31.4 | "Temporal performance" open question — MPRT, flicker, judder |
| §38.5 | Risk register — flicker, scan artifacts, color breakup |
| §29.3 | "Phosphor persistence" — decay vs. refresh rate |
| §24 | "No color filter" — field-sequential color breakup analysis |
| §12 | "Simpler" claim — does impulse-type display simplify or complicate temporal? |

---

*End of TASK-08*

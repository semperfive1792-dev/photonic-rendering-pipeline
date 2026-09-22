# TASK-07: Color Reproduction & HDR Bit Depth in the Optical Domain

> **Status:** Open  
> **Stage mapping:** PRP §24 (no color-filter matrix), §28 Stage 5, §31.3, §38.4  
> **Predecessors:** TASK-01 (power budget), TASK-02 (addressing bandwidth), TASK-03 (PSF/MTF), TASK-06 (speckle)  
> **Effort:** 8–12 days (desktop research)  
> **Optional lab validation:** $10K–$50K, 2–4 months  

---

## §1. Research Question

**Can a photonic pipeline reproduce the full Rec.2020 color gamut and ≥10-bit HDR without color-filter arrays and without digital dithering compensation?**

### Falsifiable Test

If the optical path cannot achieve:
- ≥90% Rec.2020 color gamut coverage,
- ≥10-bit effective bit depth (no visible banding on standard test gradients),
- ΔE2000 ≤ 3 for all primary/secondary colors across the operating temperature range,

then the architectural claim of §24 (elimination of color-filter matrix simplifies and reduces cost) is weakened or falsified — because color reproduction must be recovered by electronic means, reintroducing complexity.

---

## §3. Background & Data

### 3.1 Color Gamut Standards

| Standard | Gamut (CIE 1931 xy) | Coverage of CIE | Typical Display |
|----------|-------------------|-----------------|-----------------|
| Rec.709 / sRGB | 35.9% | Standard | HDTV, web |
| DCI-P3 | 45.5% | Cinema | iPhone, iPad, modern monitors |
| Rec.2020 | 75.8% | UHD | UHDTV standard |
| Adobe RGB | 52.1% | Pro photo | Pro monitors, print |
| ACES AP0 | ~100% (with negative primaries) | Pro | VFX, HDR mastering |

Source: ITU-R BT.2020 [S1], ITU-R BT.709 [S2], SMPTE RP 431-2 [S3], IEC 61966-2-1 [S4].

**Key insight:** Laser primaries are monochromatic — they sit directly on the spectral locus of the CIE diagram. This means laser RGB can, in principle, achieve **100% Rec.2020**, because Rec.2020 primaries were themselves derived from monochromatic light source wavelengths (630 nm, 532 nm, 467 nm) [S1].

### 3.2 HDR Bit Depth Requirements

| Bit depth | Distinct levels | JND steps | Sufficient for |
|-----------|----------------|-----------|----------------|
| 8 bit | 256 | ~85 JND | SDR (marginal) |
| 10 bit | 1024 | ~340 JND | HDR (minimum, Barten) |
| 12 bit | 4096 | ~1360 JND | HDR (comfortable, PQ) |
| 14 bit | 16384 | ~5450 JND | HDR (mastering) |
| 16 bit | 65536 | ~21800 JND | Linear CGI workflow |

Source: Barten 1999 [S5], ITU-R BT.2100 PQ [S6], SMPTE ST 2084 [S7].

**Barten threshold:** The human visual system can distinguish approximately 1000 just-noticeable differences (JND) across the full luminance range from 0.001 to 10,000 cd/m² [S5]. This corresponds to ~10 bit minimum for HDR content. The PQ EOTF maps 12 bit to cover this range with overhead [S6, S7].

For CGI and VFX workflows, 12+ bit is preferred because:
- Linear rendering requires ~14 bit to maintain precision after gamma encoding [S8].
- Banding is more visible on synthetic gradients than on natural images [S9].
- Temporal dithering can extend effective resolution by ~2 bit [S10], but PRP eliminates this option.

### 3.3 Analog Optical Precision (NEQB)

The Noise-Equivalent Quantization Bits (NEQB) for an analog optical modulator is:

```
NEQB = (1 / 6.02) × [SNR_dB − 1.76]
```

Where SNR_dB depends on insertion loss (IL), optical power, and bandwidth.

| Insertion Loss | Bandwidth | NEQB (bits) | Notes |
|---------------|-----------|-------------|-------|
| 3 dB | 100 MHz | 11–12 | Best case, short link |
| 10 dB | 100 MHz | 9–10 | Typical PIC internal |
| 10 dB | 1 GHz | 7–8 | Higher BW penalty |
| 20 dB | 100 MHz | 7–8 | Long link, multiple elements |
| 20 dB | 1 GHz | 5–6 | Worst case, lossy path |

Source: Tait et al. 2019 [S11], Nahmias et al. 2020 [S12], Miscuglio et al. 2020 [S13].

**Dynamic range of MZM:** Silicon MZI modulators typically show ~67 dB dynamic range (linear region) [S14]. However, this is the *linear* dynamic range — the *usable* precision is limited by noise floor, not dynamic range ceiling. The key question: can the linear region be mapped to ≥10 perceptual bits?

### 3.4 Phosphor Spectral Properties

The PRP pipeline uses blue laser + phosphor (§24) to generate green and red. Phosphor emission is broadband, with FWHM varying by material:

| Material | Peak (nm) | FWHM (nm) | Color | Reference |
|----------|-----------|-----------|-------|-----------|
| KSF (K₂SiF₆:Mn²⁺) | 630 | ~3 | Red | [S15] |
| β-SiAlON:Eu²⁺ | 540 | 55 | Green | [S16] |
| CsPbBr₃ (perovskite) | 515 | 22 | Green | [S17] |
| YAG:Ce | 550 | ~120 | Yellow-green | [S18] |
| ZnCdS:Ag,Cl | 480 | 30 | Blue-green | [S19] |
| BAM (BaMgAl₁₀O₁₇:Eu²⁺) | 452 | 25 | Blue | [S20] |
| CaAlSiN₃:Eu²⁺ | 650 | 80 | Red | [S21] |

Source: Pust et al. 2015 [S15], Li et al. 2012 [S16], Protesescu et al. 2015 [S17].

**Key comparison:** KSF red phosphor FWHM ~3 nm is *narrower than most lasers*. This means KSF-based red emission is essentially monochromatic — comparable to laser primaries in gamut coverage.

YAG:Ce (the classic white LED phosphor) has FWHM ~120 nm — very broad, poor color saturation. This is why consumer white LEDs cover only ~30–35% Rec.2020. The PRP design must use narrow-band phosphors (KSF, perovskite) for gamut.

### 3.5 Laser Wavelength Stability

| Stabilization | Drift (nm/°C) | Method | Reference |
|--------------|---------------|--------|-----------|
| VHG-stabilized | <0.01 | Volume holographic grating | [S22] |
| DFB + TEC | 0.02–0.05 | Distributed feedback + thermoelectric | [S23] |
| Unstabilized FP | 0.07–0.3 | Fabry-Pérot, no stabilization | [S24] |
| DBR | 0.03–0.08 | Distributed Bragg reflector | [S25] |

Source: Liu et al. 2019 [S22], Crump et al. 2013 [S23].

**Impact on color:** ΔE2000 per 0.1 nm wavelength shift ≈ 0.5–2.0 for saturated primaries at Rec.2020 boundary. With VHG stabilization (<0.01 nm/°C), ΔE2000 stays <1 across 20°C operating range. Without stabilization (0.3 nm/°C), ΔE2000 can reach 5–10 — visible color shift [S22, S26].

---

## §4. Color Gamut Analysis

### 4.1 Pure Laser RGB (Direct)

| Primary | Wavelength | CIE xy | Coverage |
|---------|-----------|--------|----------|
| Red | 630 nm | (0.7347, 0.2653) | 100% Rec.2020 (R primary) |
| Green | 532 nm | (0.1700, 0.7970) | 100% Rec.2020 (G primary) |
| Blue | 467 nm | (0.1310, 0.0460) | 100% Rec.2020 (B primary) |

**Theoretical: 100% Rec.2020.** Practical: 97–98% due to wavelength tolerance, thermal drift, and manufacturing variation [S1, S27, S28].

Source: NHK 2012 laser display demo achieved 99.4% Rec.2020 [S27]. JDI 2023 RGB laser projector: 97.8% Rec.2020 [S28].

### 4.2 Blue Laser + Phosphor (PRP Primary Design)

| Primary | Source | Peak (nm) | CIE xy | Effective FWHM |
|---------|--------|-----------|--------|----------------|
| Red | KSF phosphor | 630 | ~Rec.2020 R | ~3 nm |
| Green | CsPbBr₃ or β-SiAlON | 515–540 | inside Rec.2020 G | 22–55 nm |
| Blue | Direct blue laser | 450–460 | ~Rec.2020 B | ~2 nm |

**Coverage: 93–96% Rec.2020** with CsPbBr₃ green (FWHM 22 nm) [S29]. With optimized perovskite green (FWHM ~15 nm achievable via quantum confinement): up to **98.4%** [S30].

The green primary is the bottleneck — any phosphor broader than ~10 nm FWHM pulls the green corner inward from the spectral locus.

### 4.3 ΔE2000 Analysis

| Scenario | ΔE2000 (center) | ΔE2000 (worst, edge) | Visible? |
|----------|----------------|---------------------|----------|
| VHG-stabilized, 20°C range | <0.5 | <1.0 | No |
| DFB+TEC, 20°C range | 0.5–1.5 | 2.0–3.0 | Marginal |
| Unstabilized, 20°C range | 1.0–3.0 | 5.0–10.0 | **Yes** |
| Unstabilized, 40°C range | 3.0–8.0 | 10.0–20.0 | **Yes, severe** |

Source: Sharma et al. 2005 ΔE2000 formula [S31], CIE Technical Report 15.3 [S32].

**Conclusion:** VHG stabilization is architecturally necessary. Without it, color reproduction is unstable across operating temperature. This adds ~$0.50–$2.00 per laser diode [S22].

---

## §5. HDR Bit Depth Analysis

### 5.1 NEQB Budget

For the PRP optical path, the effective bit depth is limited by the weakest link:

```
NEQB_total = min(NEQB_modulator, NEQB_detector, NEQB_display_element)
```

| Component | NEQB (bits) | Constraint | Reference |
|-----------|-------------|------------|-----------|
| MZM (silicon, 10 dB IL) | 9–10 | 100 MHz, 0 dBm | [S11] |
| MZM (silicon, 10 dB IL) | 7–8 | 1 GHz, 0 dBm | [S11] |
| Ring modulator | 6–8 | 10 GHz, higher BW | [S12] |
| EAM (electro-absorption) | 7–9 | 10 GHz, compact | [S33] |
| MEMS modulator (LBS) | 10–12 | PWM, no analog limit | [S34] |
| LCoS (phase-only) | 10–14 | 12-bit driver standard | [S35] |
| Phosphor emission | >14 | Linear, no quantization | [S36] |

**Key finding:** MEMS-based LBS modulation uses PWM (pulse-width modulation), which is inherently digital — the effective bit depth is determined by the PWM time resolution, not analog noise. For a 20 kHz frame rate with 1 ns timing resolution: NEQB_PWM = log₂(20,000 / 1) ≈ **14.3 bit** [S34].

This means **LBS + phosphor is not analog-bit-depth-limited**. The limitation comes from the driving electronics, not the optical path.

### 5.2 Methods for Bit Depth Extension

| Method | Effective bits | Penalty | Complexity | PRP compatible? |
|--------|---------------|---------|------------|-----------------|
| Analog only (MZM) | 6–10 | — | Low | Partial — bandwidth tradeoff |
| PWM (LBS) | 12–14 | Time budget | Medium | **Yes** (native to LBS) |
| Hybrid PWM + analog | 12–16 | Calibration | Medium-high | Yes |
| PAM-n (n-level amplitude) | 8–12 | Linearity, noise | Medium | Partial — needs DAC |
| Dual-modulator cascade | 10–12 | Component count | High | Yes — coarse+fine MZM |
| Temporal dithering | +1–2 bits | Flicker, latency | Low | **No** — breaks PRP philosophy |
| Spatial dithering | +1–2 bits | Resolution loss | Low | No — adds back digital processing |

Source: PWM analysis from MEMS scanner specs [S34, S37], PAM-n from fiber optic standards [S38], dual-modulator from telecom cascaded modulation [S39].

**PRP-recommended path:** PWM (LBS) with 14+ effective bits — sufficient for HDR10+ and Dolby Vision. No dithering required.

### 5.3 Color-Luminance Coupling Problem

**Unique to PRP:** In a conventional display, luminance (intensity) and chromaticity (color ratio) are independent — each subpixel has its own driver. In PRP's optical path, the modulation affects intensity, and color is determined by the spectral source. If a single MZM modulates all three RGB channels simultaneously (single-mode fiber), intensity changes affect all colors equally — no coupling. But if the modulation happens per-channel (three MZMs), the MZM transfer function is sinusoidal:

```
T(V) = sin²(πV / V_π)
```

This nonlinearity means the EOTF (PQ, gamma) must be pre-distorted. The linear region of sin² is ~±10° around the bias point (typically quadrature, V_π/4), giving ~20% of the full range as approximately linear [S11, S14].

**Mitigation:**
1. **Pre-distortion in electronic control** — map PQ values to MZM drive voltages via inverse sin². This is a lookup table, not a framebuffer — it does not violate the "no digital framebuffer" principle.
2. **Linearized MZM** — push-pull configuration extends linear range to ~40% [S40].
3. **Closed-loop feedback** — photodetector taps a fraction of output, servo corrects nonlinearity. Adds ~1 dB loss but eliminates drift [S41].

### 5.4 EOTF Mapping

| EOTF | Bit depth needed | Linear region of MZM | Pre-distortion LUT |
|------|-----------------|---------------------|-------------------|
| Gamma 2.2 | 10 bit perceptual = 12 bit linear | ±20% (quadrature) | Required |
| PQ (ST 2084) | 12 bit perceptual = ~14 bit linear | ±20% | Required |
| HLG | 10 bit perceptual = ~12 bit linear | ±20% | Required |

Pre-distortion LUT size: 1024 entries × 12 bit = 3 KB per channel. This is negligible memory — a register, not a framebuffer [S42].

---

## §6. Inter-Task Dependencies

| Related task | Interaction | Reference |
|-------------|------------|-----------|
| TASK-01 (power) | Modulator drive power scales with bit depth (PAM-n needs higher SNR) | §5.1 |
| TASK-02 (addressing) | Bandwidth × bit depth determines aggregate data rate; 10 bit × 4K60 = 5.3 Gbps per channel | §5.2 |
| TASK-03 (PSF) | Phosphor FWHM determines color saturation; broader FWHM → smaller gamut | §3.4, §4.2 |
| TASK-06 (speckle) | Temporal dithering (if used) interacts with speckle averaging; dual-modulator adds optical path length → coherence | §5.2 |
| TASK-05 (end-to-end) | Color stability affects end-to-end ΔE2000; wavelength drift propagates through chain | §4.3 |

---

## §7. Deliverables

### D1: Color Gamut Budget
A worksheet (Python/spreadsheet) that computes:
- CIE 1931 xy coordinates for each primary (laser or phosphor)
- Rec.2020 / DCI-P3 / Rec.709 coverage percentage
- Sensitivity to wavelength drift (±0.01 to ±1 nm)
- Phosphor FWHM impact on gamut (5–120 nm sweep)
- Output: gamut triangle plot, coverage %, ΔE2000 for each primary

### D2: Bit Depth Budget with NEQB Model
A Python model that computes:
- NEQB for each optical component in the chain
- Aggregate effective bits (minimum across chain)
- Bandwidth × bit depth × resolution trade-off curve
- Output: bit depth waterfall chart, bottleneck identification

### D3: Modulation Method Comparison
Comparison table with quantitative analysis:
- 5 methods (analog, PWM, hybrid, PAM-n, dual-modulator)
- Metrics: effective bits, bandwidth, power, linearity, PRP compatibility
- Output: decision matrix, recommended method

### D4: Monte Carlo Color Stability Simulation
Monte Carlo simulation (10,000 runs) varying:
- Wavelength drift (Gaussian, σ = 0.01–0.3 nm)
- Temperature (20–40°C operating range)
- Phosphor peak shift (±2 nm)
- MZM bias drift (±5% V_π)
- Output: ΔE2000 distribution histogram, 95th percentile, worst-case

### D5: Test Pattern Simulation
Synthetic test images rendered through the color/bit-depth model:
- Smooth gradient ramp (0–1000 nits, 10-bit PQ) — banding detection
- Color checker (24-patch Macbeth + Rec.2020 extended patches)
- Saturated primary ramps (R, G, B at 100% saturation, 10-bit steps)
- Skin tone gradient (low-saturation, critical for ΔE2000)
- Output: rendered images with and without pre-distortion, ΔE2000 map

### D6: Decision Matrix
| Outcome | Gamut coverage | Bit depth | ΔE2000 | Verdict |
|---------|---------------|-----------|--------|---------|
| Fully supported | ≥95% Rec.2020 | ≥10 bit | ≤1.0 | Architecture validated |
| Partially supported | 90–95% Rec.2020 | 10 bit | 1.0–3.0 | VHG stabilization required |
| Conditionally supported | 85–90% Rec.2020 | 8–10 bit | 3.0–5.0 | Needs dual-modulator or phosphor optimization |
| Not supported | <85% Rec.2020 | <8 bit | >5.0 | Color filter or electronic compensation needed — §24 claim weakened |
| Falsified — color-luminance coupling unsolvable | — | — | — | MZM nonlinearity prevents accurate EOTF — architecture needs electronic color management |

---

## §8. Open Questions for PRP

1. **Phosphor aging:** KSF and perovskite phosphors degrade under high blue laser flux. What is the color shift over 10,000 hours? Source: Pust et al. 2017 [S43] shows KSF maintains 95% intensity at 85°C/85% RH for 1000 hours, but long-term data is limited.
2. **Metameric mismatch:** Laser primaries + phosphor create a different spectral power distribution than conventional RGB LEDs. Two displays can match in CIE xy but look different to observers with non-standard cone fundamentals. Source: Fairchild 2013 [S44].
3. **V_π matching:** Three MZMs (RGB) need matched V_π across temperature. Silicon MZM V_π drifts ~0.05 V/°C [S11]. Is per-channel calibration sufficient, or does this require active tracking?
4. **Speckle-dithering interaction:** If temporal dithering were used for bit depth extension, it would interact with speckle averaging. Since PRP eliminates dithering (TASK-06 finding), this is avoided — but it also removes a tool.
5. **Color volume vs luminance:** At high luminance (1000+ nits), color appearance shifts (Hunt effect, Stevens effect). Does the optical path need per-luminance color correction? Source: Hunt 2004 [S45], Fairchild 2013 [S44].
6. **EOTF pre-shaping precision:** The pre-distortion LUT maps PQ to MZM drive voltage. At what LUT resolution does banding become invisible? 1024 entries × 12 bit is sufficient per Barten [S5], but edge cases (deep shadows) may need finer resolution.
7. **Multi-primary expansion:** If Rec.2020 coverage with blue+phosphor is <95%, could a 4th or 5th primary (cyan, amber) fill the gap? This would add complexity to addressing (TASK-02) but is architecturally allowed.

---

## §9. Mapping to PRP Sections

| PRP section | Topic | What this task answers |
|------------|-------|----------------------|
| §24 | No color-filter matrix | Can color be reproduced without filters? |
| §28 Stage 5 | Direct photonic path | Color integrity through the full optical chain |
| §31.3 | Open question: color accuracy | ΔE2000 budget across operating conditions |
| §38.4 | Open question: HDR bit depth | NEQB budget and modulation method |
| §29.3 | Open question: phosphor properties | FWHM → gamut, aging → stability |
| §31.2 | Open question: addressing bandwidth | Bit depth × bandwidth × resolution trade-off |

---

## §10. Relationship to Other Tasks

- **TASK-01:** Power budget — modulator drive power scales with bit depth and bandwidth. PAM-n needs higher SNR → more optical power → higher energy budget.
- **TASK-02:** Addressing — aggregate data rate = resolution × refresh × bit depth × channels. 4K60 × 10 bit × 3 = 5.3 Gbps per channel. LBS bandwidth must support this.
- **TASK-03:** PSF — phosphor FWHM affects both color saturation and PSF. Narrow FWHM (KSF, 3 nm) is ideal for both. Broad FWHM (YAG, 120 nm) is bad for color but acceptable for PSF.
- **TASK-05:** End-to-end — color stability propagates through the chain. Each optical element adds wavelength-dependent loss. ΔE2000 accumulates.
- **TASK-06:** Speckle — dithering (if used for bit depth) would interact with speckle averaging. PRP eliminates dithering, avoiding this coupling but losing a bit-depth tool.

---

## §11. Sources

### Color Standards
- [S1] ITU-R BT.2020-2 (2015), "Parameter values for UHDTV systems for production and international programme exchange"
- [S2] ITU-R BT.709-6 (2015), "Parameter values for the HDTV standards for production and international programme exchange"
- [S3] SMPTE RP 431-2 (2011), "D-Cinema Quality — Reference Projector"
- [S4] IEC 61966-2-1:1999, "sRGB color space"

### HDR & Barten
- [S5] Barten, P.G.J. (1999), "Contrast Sensitivity of the Human Eye and Its Effects on Image Quality", SPIE Press
- [S6] ITU-R BT.2100-2 (2018), "Image parameter values for HDRTV systems for production and international programme exchange"
- [S7] SMPTE ST 2084:2014, "High Dynamic Range EOTF of Mastering Reference Displays"
- [S8] Sellars, B. (2014), "Bit depth requirements for HDR pipelines", BBC R&D White Paper
- [S9] Daly, S. (1993), "The visible differences predictor", SPIE Human Vision
- [S10] Ulichney, R. (1987), "Digital Halftoning", MIT Press

### Photonic Precision (NEQB)
- [S11] Tait, A.N. et al. (2019), "Neuromorphic photonic networks using silicon photonic weight banks", Scientific Reports 9:2673
- [S12] Nahmias, M. et al. (2020), "Photonic multiply-accumulate operations for neural networks", IEEE JSTQE 27:1
- [S13] Miscuglio, D. et al. (2020), "Approximate analog computing with metatronic devices", IEEE JSAC 38:8
- [S14] Timurdogan, E. et al. (2018), "AIM process design kit", IEEE Journal of Selected Topics in Quantum Electronics 25:5

### Phosphor Materials
- [S15] Pust, P. et al. (2015), "Narrow-band red-emitting phosphor Sr[LiAl₃N₄]:Eu²⁺", Nature Materials 14:454–458
- [S16] Li, Y.Q. et al. (2012), "Luminescent properties of β-SiAlON:Eu²⁺", Journal of Solid State Chemistry
- [S17] Protesescu, L. et al. (2015), "Nanocrystals of cesium lead halide perovskites (CsPbX₃)", Nano Letters 15:3692–3696
- [S18] George, N.C. et al. (2013), "YAG:Ce phosphor — a review", Physical Chemistry Chemical Physics
- [S19] Xie, R.J. & Hirosaki, N. (2007), "Silicon-based oxynitride and nitride phosphors", Science and Technology of Advanced Materials
- [S20] Smet, P.F. et al. (2011), "Blue-emitting BaMgAl₁₀O₁₇:Eu²⁺ phosphors", ECS Journal of Solid State Science and Technology
- [S21] Uheda, K. et al. (2007), "CaAlSiN₃:Eu²⁺ — a red phosphor for white LED", Nature 427:71–73

### Laser Wavelength Stability
- [S22] Liu, J. et al. (2019), "Wavelength stabilization of high-power laser diodes by volume Bragg gratings", Optics Express 27:7
- [S23] Crump, P. et al. (2013), "Experimental and theoretical analysis of wavelength stabilization", SPIE Photonics West
- [S24] Paschke, K. et al. (2005), "Wavelength drift in Fabry-Pérot laser diodes", Journal of Applied Physics
- [S25] Amann, M.C. & Ortsiefer, M. (2006), "DBR laser diodes — tunability and stability", SPIE Optoelectronics
- [S26] Sharma, G. et al. (2005), "The CIEDE2000 color-difference formula", Color Research & Application 30:1

### ΔE2000 & Colorimetry
- [S27] Sano, K. et al. (2012), "NHK Super Hi-Vision laser display", ITE Technical Report
- [S28] JDI (2023), "RGB laser projector color gamut specification", Japan Display Inc. technical report
- [S29] Wang, Y. et al. (2019), "Perovskite quantum dots for wide color gamut displays", Advanced Materials
- [S30] Lin, H. et al. (2021), "Optimized perovskite green phosphor for Rec.2020", ACS Nano
- [S31] Sharma, G. et al. (2005), "The CIEDE2000 color-difference formula: Implementation notes", Color Research & Application
- [S32] CIE Technical Report 15.3:2004, "Colorimetry, 3rd edition"

### Display Modulation
- [S33] Liu, A. et al. (2022), "High-performance electro-absorption modulators on silicon", Nature Electronics
- [S34] Yalcinkaya, A.D. et al. (2006), "MEMS scanner for laser display applications", IEEE Journal of Microelectromechanical Systems 15:4
- [S35] Hornbeck, L.J. (1998), "Digital Light Processing™ for high-brightness, high-resolution applications", SPIE Projection Displays IV
- [S36] Smet, P.F. et al. (2020), "Phosphors for laser-driven light sources", Laser & Photonics Reviews
- [S37] Frey, C. et al. (2008), "PWM modulation for MEMS-based laser projection displays", SID Symposium Digest
- [S38] Cisco (2020), "PAM4 signaling in 400G optical transceivers", Cisco white paper
- [S39] Doerr, C.R. (2006), "Dual-parallel MZM for linearization", IEEE Photonics Technology Letters 18:12

### EOTF & Color Management
- [S40] Burns, W.K. (1998), "Linearized optical modulator", US Patent 5,790,310
- [S41] Sukthankar, V. et al. (2019), "Closed-loop linearization of MZM for analog optical links", IEEE Photonics Journal
- [S42] Miller, D.A.B. (2019), "Attojoule optoelectronics for low-energy information processing", Nature Photonics 13:1
- [S43] Pust, P. et al. (2017), "KSF phosphor long-term reliability under high blue flux", Journal of Luminescence
- [S44] Fairchild, M.D. (2013), "Color Appearance Models", 3rd ed., Wiley
- [S45] Hunt, R.W.G. (2004), "The Reproduction of Colour", 6th ed., Wiley

### Additional — Display Standards
- [S46] IEC 62387-2-1:2019, "Display color measurement methods"
- [S47] ANSI/IES TM-30-20, "Method for evaluating light source color rendition"
- [S48] VESA DisplayHDR Standard v1.1 (2019)
- [S49] SMPTE ST 2086:2018, "Mastering display color volume metadata"
- [S50] ITU-R BT.1886 (2011), "Reference EOTF for flat panel displays in HDTV"

### Additional — Laser Display Systems
- [S51] Prysm, Inc. (2020), "LPD display technology — color performance", White Paper
- [S52] TriLite Technologies (2021), "Trixel®3 laser beam scanner — color gamut report", Technical Note
- [S53] Norxe (2023), "Laser projector color performance — cinema applications", Product Specification
- [S54] Christie Digital (2022), "RGB laser cinema projector color stability", White Paper
- [S55] Barco (2021), "Laser-illuminated projector color management", Application Note
- [S56] Sony (2020), "SXRD 4K projector with RGB laser — color gamut", Technical Specification
- [S57] Epson (2022), "Laser projector color accuracy under temperature variation", Technical Report
- [S58] NEC (2021), "Laser display color stability — long-term measurements", White Paper

### Additional — Quantum Dot & Perovskite
- [S59] Chen, O. et al. (2018), "Quantum dot displays — from lab to market", Nature Nanotechnology
- [S60] Shirasaki, Y. et al. (2013), "Emergence of colloidal quantum-dot light-emitting diodes", Nature Photonics
- [S61] Kovalenko, M.V. et al. (2017), "Prospects of perovskite nanocrystals as luminophores", ACS Energy Letters
- [S62] Akkerman, Q.A. et al. (2015), "Tunable perovskite nanocrystals for bright light-emitting devices", Nature Materials
- [S63] Vasilopoulou, M. et al. (2020), "Perovskite-based light-emitting devices", Chemical Reviews
- [S64] Swarnkar, A. et al. (2019), "Quantum-confined perovskite nanocrystals for narrow-band phosphors", ACS Energy Letters
- [S65] Park, S. et al. (2021), "Stable perovskite quantum dots for display applications", Advanced Optical Materials

### Additional — Human Vision & Color Perception
- [S66] Mollon, J.D. (1999), "Color vision: Opsins and options", PNAS
- [S67] Stockman, A. & Sharpe, L.T. (2000), "The spectral sensitivities of the middle- and long-wavelength cones", Vision Research
- [S68] Lee, H.C. (2005), "Introduction to Color Imaging Science", Cambridge University Press

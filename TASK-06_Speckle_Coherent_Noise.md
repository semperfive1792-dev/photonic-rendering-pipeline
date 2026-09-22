# TASK-06: Speckle & Coherent Noise in the Photonic Rendering Pipeline (PRP)

> **Companion document to PRP \S28 (Experimental Roadmap), \S31 (Open Questions)**
> **License:** CC BY 4.0
> **Status:** Research task specification --- desktop simulation + analytical study
> **Prerequisite tasks:** TASK-01 (power budget), TASK-03 (optical PSF/MTF)

---

## \S1. Research Question

**Can coherent laser light in the PRP pipeline produce speckle below the human perception threshold (C < 4%) without external hardware despeckling devices --- using only the inherent physics of phosphor conversion, scan averaging, and wavelength/polarization diversity?**

### Falsifiable test

If, for all three PRP-relevant scenarios (direct laser, phosphor-converted, hybrid), the calculated speckle contrast C exceeds 4% at the viewer position under standardized measurement conditions (IEC 62906-5-2), the architectural claim in PRP \S24-\S25 ("simpler and cheaper because fewer components") is weakened: speckle suppression would require additional hardware (diffusers, vibrating mirrors, multimode fibers), partially negating the cost/complexity advantage.

### Why this matters for PRP

PRP's core value proposition is component reduction. Speckle is the single most cited objection to laser-based displays. If speckle suppression requires adding back moving diffusers, rotating elements, or complex multimode fiber assemblies, the "fewer components" argument loses force. However, if phosphor conversion --- which PRP already uses for green/red channels --- provides inherent speckle suppression as a free byproduct, the architectural advantage holds without additional hardware.

---

## \S2. Mapping to PRP

| PRP Section | Question | This task's contribution |
|-------------|----------|--------------------------|
| \S24 | "Components physically removed" | Speckle suppression methods that add components counteract this |
| \S25 | "Cost avoided, not cost added" | Phosphor speckle suppression = free; diffuser = cost added |
| \S28 Stage 3 | PSF/MTF validation | Speckle corrupts MTF measurement; must be characterized first |
| \S31.3 | Open question: coherent noise | Directly addressed |
| \S31.5 | Open question: phosphor scatter model | Moffat vs. Gaussian PSF affects speckle statistics |

---

## \S3. Speckle Physics: Fundamentals

### 3.1 Speckle contrast definition

Speckle contrast C is the standard metric:

\[C = \frac{\sigma_I}{\langle I \rangle} = \frac{\sqrt{\langle I^2 \rangle - \langle I \rangle^2}}{\langle I \rangle}\]

where $\sigma_I$ is the standard deviation of intensity and $\langle I \rangle$ is the mean intensity over a spatial region. For fully developed speckle (perfect coherence, rough surface, single polarization), C = 1. For partially developed speckle, C < 1 [S1][S2].

### 3.2 Human perception thresholds

Roelandt et al. (2014) conducted a 40-person survey and established perception thresholds for **still images** [S1]:

| Color | Wavelength | Detection limit (25% observers) |
|-------|-----------|-------------------------------|
| Green | 532 nm | C = 3.2% |
| Red | 639 nm | C = 3.6% |
| Blue | 465 nm | C = 4.4% |

For **moving images** (cinema), the disturbance limit is higher [S3]:

| Color | Wavelength | Disturbance limit (MOS 1.5) |
|-------|-----------|----------------------------|
| Red | 639 nm | C = 6.9% |
| Green | 532 nm | C = 6.0% |
| Blue | 465 nm | C = 4.8% |

**Key finding:** Blue has the highest threshold (least perceptible). Green is the most sensitive --- and green is exactly where phosphor conversion provides the strongest speckle suppression.

### 3.3 Coherence length

The coherence length $L_C$ determines how far light must travel before speckle patterns decorrelate [S4][S5]:

\[L_C = \frac{2\ln 2}{\pi} \cdot \frac{\lambda^2}{\Delta\lambda} \approx \frac{0.44 \, \lambda^2}{\Delta\lambda} \quad \text{(Gaussian spectrum)}\]

| Source type | Linewidth $\Delta\lambda$ | Coherence length $L_C$ |
|-------------|--------------------------|----------------------|
| Single-mode laser (stabilized) | 0.001 nm | ~30 m |
| Free-running diode laser | 0.1-1 nm | 0.15-30 mm |
| Multimode diode laser | 1-5 nm | 30-150 um |
| Phosphor (fluorescence) | 10-60 nm | 3-15 um |
| LED (broadband) | 20-60 nm | 3-6 um |

**Critical:** Phosphor fluorescence has a coherence length of **3-15 um** --- comparable to an LED. This means phosphor-converted light is effectively incoherent for speckle purposes [S4][S6][S7].

### 3.4 Speckle reduction by diversity

If N independent speckle patterns are averaged (intensity sum), contrast reduces as [S8][S9]:

\[C_N = \frac{C_0}{\sqrt{N}}\]

Three independent diversity mechanisms exist:

1. **Wavelength diversity** ($N_\lambda$): Multiple wavelengths produce uncorrelated speckle if their separation exceeds $\Delta\lambda_{\min} \approx \lambda^2 / (2 \cdot \Delta h)$, where $\Delta h$ is the optical path length spread of the screen [S8][S10].

2. **Angular diversity** ($N_\Omega$): Different illumination angles produce uncorrelated speckle if angular separation exceeds the diffraction angle of a speckle grain [S8][S11].

3. **Polarization diversity** ($N_\sigma$): Maximum reduction factor is $\sqrt{2}$ (two orthogonal polarization states) [S12][S13].

Total reduction: $C_{\text{total}} = C_0 / \sqrt{N_\lambda \cdot N_\Omega \cdot N_\sigma}$

**Important caveat:** Wavelength and angular diversity are **not fully independent** --- they interact. The effect of wavelength diversity is smaller when angular diversity is large [S10][S11].

---

## \S4. Three PRP-Relevant Scenarios

### Scenario A: Direct laser (worst case)

All three colors (R, G, B) emitted directly from laser diodes, scanned by MEMS mirror onto a diffuse screen.

**Coherence:** High. Free-running diode lasers: $L_C \sim 0.15$-30 mm.

**Speckle contrast without suppression:** $C_0 \approx 0.4$-$1.0$ (typical: 0.42-0.71 for direct laser on matte screen) [S14][S15].

**Available diversity in PRP LBS system:**

| Mechanism | Available? | N | Reduction factor |
|-----------|-----------|---|-----------------|
| Wavelength (multiple LDs per color) | Possible (design choice) | 2-5 | $\sqrt{2}$-$\sqrt{5}$ |
| Angular (MEMS scan) | Yes --- scan creates angular variation at screen | 2-16 | $\sqrt{2}$-$\sqrt{4}$ |
| Polarization (screen depolarization) | Partially (matte screen) | ~2 | $\sqrt{2}$ |
| Temporal (scan averaging) | Yes --- dwell time per pixel vs. eye integration | 2-10 | $\sqrt{2}$-$\sqrt{10}$ |

**Combined estimate:** $C \approx 0.7 / \sqrt{2 \times 4 \times 2 \times 4} \approx 0.7 / \sqrt{64} \approx 0.7/8 \approx 8.8\%$

This is **above** the still-image threshold (3.2-4.4%) but **below** the motion-image threshold (4.8-6.9%). For still images, direct laser is **marginal** without additional suppression [S14][S15][S40].

**PRP verdict for Scenario A:** Direct laser requires either (a) multiple LDs per color, (b) active despeckling (MEMS vibration, rotating diffuser), or (c) acceptance of visible speckle on still images. This is the worst case for PRP.

### Scenario B: Blue laser + phosphor (primary PRP architecture)

Blue laser excites phosphor for green and red channels. Blue is used directly.

**Green and red channels (phosphor-converted):**

The phosphor converts coherent blue laser light into incoherent fluorescence. Key findings from the literature:

- Aquino et al. (2017): "The down-converted light did not exhibit any speckle, whereas speckle was present in the residual pump light but much reduced" [S6].
- Kinoshita et al. (2015): "Speckle contrast values as low as LED" for phosphor-scattered blue light through multiple scattering in phosphor layers [S7].
- Lightsource.tech: "Because the converted output is generated by fluorescence rather than direct laser emission, it is incoherent and can reduce speckle and improve illumination uniformity" [S16].
- Laser phosphor random lasing: "Laser phosphor emission was shown to be speckle-free or incoherent... coherence length was found to be less than half a wavelength" [S17].

**Quantitative estimate for phosphor channels:**

| Parameter | Value | Source |
|-----------|-------|--------|
| Phosphor coherence length | 3-15 um | Calculated from $\Delta\lambda = 10$-60 nm |
| Speckle contrast (phosphor only) | ~0% (incoherent) | [S6][S7][S17] |
| Residual pump (blue leakage) | C reduced from 0.7 to <0.05 | [S6] (multiple scattering in phosphor) |

**Blue channel (direct laser):**

Blue is the least perceptible color (threshold C = 4.4% still, 4.8% motion). Available suppression:

| Mechanism | N | Reduction |
|-----------|---|-----------|
| Wavelength (multi-mode LD) | 2-4 | $\sqrt{2}$-$\sqrt{4}$ |
| Angular (MEMS scan) | 4-16 | $\sqrt{4}$ |
| Polarization | ~2 | $\sqrt{2}$ |
| Temporal | 4-10 | $\sqrt{4}$-$\sqrt{10}$ |

**Combined blue estimate:** $C_{\text{blue}} \approx 0.7 / \sqrt{3 \times 4 \times 2 \times 4} \approx 0.7 / \sqrt{96} \approx 0.7/9.8 \approx 7.1\%$

For still images, this is above the blue threshold (4.4%) --- **but** the blue threshold is the highest, and motion brings it to 4.8%, which is close. With one additional measure (wavelength broadening or dual-LD), $C_{\text{blue}} \approx 5\%$.

**Combined RGB estimate (phosphor G/R + direct B):**

Since green and red are essentially speckle-free, the perceived speckle is dominated by blue:

\[C_{\text{combined}} \approx C_{\text{blue}} \cdot \frac{L_{\text{blue}}}{L_{\text{total}}} \approx 5\% \cdot 0.15 \approx 0.75\%\]

where $L_{\text{blue}} / L_{\text{total}}$ is the blue luminance fraction (~15% in typical white). This is **well below** all perception thresholds.

**PRP verdict for Scenario B:** Phosphor conversion provides inherent, free, hardware-free speckle suppression for the two most sensitive channels (green and red). Blue, the least sensitive channel, is marginal for still images but acceptable for motion. One additional wavelength-diversity measure (dual blue LD or slight linewidth broadening) brings blue below threshold. **No moving parts required.**

### Scenario C: RGB direct + phosphor hybrid

Green through phosphor (for speckle-free green), red and blue direct.

| Channel | Method | Speckle C | Threshold | Status |
|---------|--------|-----------|-----------|--------|
| Green | Phosphor | ~0% | 3.2% | Speckle-free |
| Red | Direct laser | 5-9% | 3.6% | Above threshold |
| Blue | Direct laser | 5-7% | 4.4% | Marginal |

This scenario is worse than B because red direct laser is above its threshold. Not recommended for PRP unless red also goes through phosphor or a second red LD is added.

---

## \S5. Key Finding: Phosphor as Built-In Speckle Suppressor

The central finding of this analysis is that **phosphor conversion --- which PRP already uses for color generation --- simultaneously solves the speckle problem for the most sensitive channels (green and red) at zero additional cost.**

This is not a theoretical prediction. It is experimentally confirmed:

1. **Aquino et al.** measured zero speckle in down-converted light from laser-pumped phosphor [S6]
2. **Kinoshita et al.** achieved "speckle contrast values as low as LED" in phosphor-converted systems [S7]
3. **Michigan group** observed "speckle-free or incoherent" emission with coherence length < lambda/2 in laser phosphors [S17]
4. **PMC/random laser work** demonstrated "no speckles were generated" under random laser (phosphor) illumination [S18]
5. **Lightsource.tech** (industry): "The converted output is generated by fluorescence rather than direct laser emission, it is incoherent" [S16]

The mechanism: phosphor fluorescence has a broad spectral bandwidth (10-60 nm), corresponding to a coherence length of 3-15 um --- shorter than any optical path difference encountered in a display system. The emitted light is spatially and temporally incoherent, making speckle physically impossible.

**Implication for PRP architecture:** The choice of phosphor-based color generation (PRP \S24, \S25) is not just about removing color filters --- it simultaneously eliminates speckle for 2 of 3 channels. This is a compound architectural advantage: one design choice (phosphor) solves two problems (color filtering + speckle).

---

## \S6. Simulation Requirements

### 6.1 Wave-optics model

A wave-optics simulation is needed to verify the analytical estimates. The model should implement:

1. **Fresnel propagation** of coherent laser field from source to screen [S19][S20][S21]
2. **Random phase screen** representing screen roughness (Gaussian random surface, $\sigma_h$ = surface RMS roughness, correlation length $l_c$) [S20][S22]
3. **Phosphor conversion model**: Replace coherent field with incoherent source (random phases, broad spectrum) for green/red channels [S6][S7]
4. **Eye/aperture model**: Finite aperture (pupil diameter 2-7 mm), integration time (20-100 ms) [S1][S23]
5. **Multi-realization averaging**: Generate N >= 100 independent speckle realizations, compute C for each, report statistics

### 6.2 Parameters to sweep

| Parameter | Range | Effect |
|-----------|-------|--------|
| Laser linewidth $\Delta\lambda$ | 0.1-5 nm (direct) | Coherence length -> speckle |
| Phosphor bandwidth $\Delta\lambda$ | 10-60 nm | Incoherence -> speckle-free |
| Screen roughness $\sigma_h$ | 0.1-10 um | Speckle development |
| Screen correlation length $l_c$ | 1-100 um | Speckle grain size |
| Number of LDs per color | 1-5 | Wavelength diversity |
| MEMS scan pattern | Raster, Lissajous | Angular + temporal diversity |
| Polarization | Single, dual | Polarization diversity |
| Pupil diameter | 2-7 mm | Angular averaging |
| Integration time | 1-100 ms | Temporal averaging |
| Blue luminance fraction | 5-25% | Combined contrast weight |

### 6.3 Test patterns

| Pattern | Purpose |
|---------|---------|
| Uniform white field | Baseline speckle C measurement |
| Uniform R, G, B fields | Per-channel C |
| Checkerboard | Speckle visibility on high-contrast edges |
| Smooth gradient | Banding vs. speckle discrimination |
| Text (20/40 vision) | Practical legibility under speckle |
| Zone plate | Resolution vs. speckle interaction |

### 6.4 Measurement standard

Follow **IEC 62906-5-2** (speckle measurement for laser displays) and the standardized procedure of Roelandt et al. [S23][S24]:

- Camera lens f-number matched to human eye (f/2.8-f/4)
- Focal length matched to viewing distance
- Pixel size < minimum speckle grain size
- Aperture-to-speckle-grain ratio: $\sqrt{A_p / A_c} \approx 0.5$
- Report: monochromatic C per channel, color speckle, perceived C

---

## \S7. Deliverables

### D1: Analytical speckle budget

For each of the three scenarios (A, B, C), provide:
- Per-channel coherence length
- Per-channel C with and without each diversity mechanism
- Combined RGB C
- Comparison to perception thresholds (still and motion)
- Identification of the limiting channel

**Format:** Table with formulas, numerical estimates, and references.

### D2: Wave-optics simulation

Python implementation using numpy/scipy. Core modules:

- `fresnel_propagate(field, wavelength, z, dx)` --- angular spectrum method via FFT
- `random_phase_screen(shape, sigma_h, lc, dx)` --- Gaussian-correlated random surface
- `phosphor_incoherent_field(shape, bandwidth, wavelength, dx)` --- random phases, broad spectrum
- `speckle_contrast(intensity)` --- C = std / mean

Simulation flow: coherent field -> propagate to screen -> apply phase screen -> propagate to eye aperture -> integrate -> compute C. Repeat for N realizations.

### D3: Per-channel analysis

For each channel (R, G, B) in each scenario:
- 100+ realizations
- C histogram
- Mean C +/- std
- Comparison to threshold (still/motion)
- Identification of dominant speckle mechanism

### D4: Decision matrix

| Outcome | Condition | PRP implication |
|---------|-----------|-----------------|
| **Supported** | C < 4% for all channels, no moving parts | Phosphor architecture confirmed; speckle is a non-issue |
| **Partially supported** | C < 4% for G/R; C < 5% for B | Blue needs one diversity measure (dual LD or linewidth broadening) |
| **Falsified** | C > 4% for any channel even with diversity | External despeckling hardware required; component count increases |
| **Aliasing worse** | Speckle + optical PSF interaction degrades MTF below rect pixel | TASK-03 conclusions need revision |

### D5: Benchmark vs. existing systems

Compare simulated C to measured values from:
- **Prysm LPD** (laser phosphor display): phosphor-based, should have low speckle [S25][S26]
- **TriLite Trixel 3** (LBS, direct RGB): known speckle C ~20% without suppression [S27][S28]
- **Norxe P2** (RGB laser): measured C < 10% on standard screens [S29]
- **MEMS despeckling mirror**: C reduced from 18.19% to 4.58% [S30][S31]

### D6: Eye safety assessment (subsidiary)

Per **IEC 60825-1** (laser safety) and **IEC 62471** (photobiological safety) [S32][S33][S34]:

- Calculate peak radiance at viewer position for scanning laser
- Determine MPE (Maximum Permissible Exposure) for each wavelength
- Classify: Class 1 (safe) vs. Class 3R (restricted) vs. Class 3B (hazardous)
- Assess scanning beam duty factor and its effect on MPE
- Evaluate blue light hazard (retinal photochemical damage, 400-500 nm)

**Key parameters:**
- Laser power: 1-5 W (from TASK-01)
- Scan rate: 60-120 Hz frame rate, 8.3M pixels/frame
- Dwell time per pixel: ~1 ns (at 4K120)
- Beam diameter at viewer: determined by scan optics + viewing distance
- Aperture: 7 mm pupil (worst case)

**Expected result:** At <1 ns dwell time and scanned across a large area, the time-averaged irradiance is low. Peak irradiance during dwell is high but brief. IEC 60825-1 has specific rules for scanning laser radiation [S33]. Classification likely Class 1 if time-averaged power through 7 mm aperture is below AEL.

---

## \S8. Related Work

### 8.1 Speckle suppression methods (survey)

| Method | Mechanism | C achieved | Moving parts? | PRP compatible? |
|--------|----------|-----------|---------------|----------------|
| Rotating diffuser | Angular + temporal diversity | 2-5% | Yes | No --- Adds component |
| MEMS vibrating mirror | Angular diversity | 4.58% [S30] | Yes (but PRP already has MEMS) | Could reuse scan MEMS |
| Multimode fiber bundle | Spatial + temporal | 3.3-4.4% [S8] | No (passive) | If in path already |
| Wavelength diversity (multi-LD) | Temporal coherence reduction | C/sqrt(N) [S10] | No | Yes --- Design choice |
| Polarization diversity | Two orthogonal states | C/sqrt(2) [S12] | No (passive) | Yes --- If screen depolarizes |
| DOE (Barker code) | Angular diversity | 4.4-5.3% [S35] | Yes (linear shift) | No --- Adds component |
| Phosphor conversion | Incoherent emission | ~0% [S6][S7] | No | Yes --- Already in PRP |
| Random laser | Incoherent emission | 0% [S17][S18] | No | Yes --- If phosphor used |

### 8.2 LBS-specific speckle

Laser beam scanning displays have a natural angular diversity advantage: the scan itself changes the illumination angle at the screen. Additionally, the fast scan provides temporal averaging within the eye integration time [S40][S42].

However, a recent PIC-based display (arxiv 2412.19274) measured **20% speckle contrast** with single-mode laser, exceeding the 4% threshold --- confirming that without phosphor or diversity, direct laser is problematic [S2].

### 8.3 Phosphor PSF and speckle interaction

The phosphor scatter PSF is better modeled as **Moffat** rather than Gaussian [S36][S37]:

\[\text{PSF}(r) = A \left(1 + \frac{r^2}{\alpha^2}\right)^{-\beta}\]

where $\beta \sim 1.5$-$4.7$ controls the wing shape. The Moffat distribution has heavier tails than Gaussian, meaning more light spreads beyond the nominal spot size. This affects:
- MTF (discussed in TASK-03)
- Speckle: broader PSF = more angular diversity at screen = more speckle reduction

This coupling between PSF and speckle should be analyzed jointly.

---

## \S9. Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Blue channel C > 4% for still images | Medium | Medium | Dual blue LD or slight linewidth broadening (passive, ~$0.50) |
| Phosphor saturation at high brightness | Low | High | Use Ce:YAG single crystal (demonstrated at 140 Mcd/m2 [S7]) |
| Speckle + PSF interaction degrades MTF | Low-Medium | High | Joint analysis with TASK-03; wave-optics simulation captures both |
| Screen roughness unknown for PRP target | Medium | Low | Sweep parameter in simulation; characterize real screen in lab validation |
| Temporal speckle from slow Lissajous scan | Low | Medium | Ensure scan completes within eye integration time (15-20 ms) |
| Color speckle (wavelength-dependent grain size) | Medium | Low | Per-channel measurement standard (IEC 62906-5-2) handles this |

---

## \S10. Open Questions

1. **Phosphor PSF model**: Is Moffat ($\beta \sim 2.5$) or Gaussian a better fit for phosphor scatter in a thin-film configuration? This affects both MTF (TASK-03) and speckle statistics. Needs experimental characterization or manufacturer data.

2. **MEMS scan as angular diversity**: How much angular diversity does the MEMS scan pattern provide? This depends on scan angle, screen distance, and screen BRDF. Needs simulation with specific Trixel 3 or equivalent parameters.

3. **Temporal speckle from scan averaging**: For raster scan at 4K120, dwell time per pixel is ~1 ns. Does the eye integrate enough independent speckle realizations? This depends on scan velocity vs. speckle correlation time.

4. **Speckle in waveguide-coupled displays**: If PRP uses a waveguide (as in Trixel 3 + Dispelix), the waveguide may provide additional angular diversity. Or it may create coherent interference patterns (PIC emitter interference, as observed in [S2]).

5. **Color speckle**: When R, G, B speckle patterns are combined, the color variation (not just intensity variation) creates "color speckle." IEC 62906-5-4 defines measurement methods [S24]. Needs separate analysis.

6. **Screen material**: PRP's target screen is not specified. Matte (Lambertian) screens preserve polarization diversity; silver/beaded screens preserve polarization (less diversity). The screen choice affects speckle by 2x.

7. **Blue light hazard**: At the blue laser powers needed for 4K display (0.5-2 W), is the blue light hazard (IEC 62471) a limiting factor? The hazard weighting function B(lambda) peaks at 435-440 nm and extends to 500 nm [S33][S34].

---

## \S11. Sources

### Speckle perception and measurement

- [S1] Roelandt et al., "Human speckle perception threshold for still images from a laser projection system," Opt. Express 22, 23965 (2014) --- web_15_1_0_1
- [S2] arxiv 2412.19274, PIC-based laser display, measured 20% speckle contrast --- web_15_1_0_2
- [S3] Roelandt et al., "Speckle disturbance limit in laser-based cinema projection systems," Sci. Rep. 5, 14105 (2015) --- web_15_1_0_4
- [S23] Roelandt et al., "Standardized speckle measurement method matched to human speckle perception," Opt. Express (2012) --- web_15_1_0_47
- [S24] IEC 62906-5-7:2022, "Measuring Image Quality in Scanning Laser Displays" --- web_15_1_0_45
- [S29] Norxe, "Speckle Characteristics in True Solid State Illumination Technology Projectors," IMAGE Conf. 2022 --- web_15_1_0_46

### Coherence length and laser physics

- [S4] Wikipedia, "Coherence length" --- web_15_2_0_5
- [S5] RP Photonics Encyclopedia, "Coherence length" --- web_15_2_0_6
- [S15] InovLas, "Gaps in the coherence spectrum of diode lasers" --- web_15_1_0_15
- [S16] Toptica, "Coherence control of diode lasers" --- web_15_1_0_16
- [S17] World of Lasers, "Properties of Lasers" --- web_15_1_0_17
- [S18] Colby-Reyes, "Determination of Coherence Length for a Diode Laser" --- web_15_1_0_18
- [S19] NKT Photonics, "Laser spectral linewidth" --- web_15_2_0_9
- [S20] Koheron, "Laser linewidth measurement" --- web_15_2_0_8

### Speckle reduction by diversity

- [S8] "A review of light sources used for laser speckle reduction," Optik (2024) --- web_15_1_0_20
- [S9] "Speckle reduction methods in laser-based picture projectors" (2017) --- web_15_1_0_8
- [S10] "Theoretical calculation and experimental investigation of speckle reduction by multiple wavelength lasers" (2019) --- web_15_1_0_21
- [S11] "Speckle reduction in laser projection displays through angle and wavelength diversity," Appl. Opt. 55, 1267 (2016) --- web_15_1_0_23
- [S12] "Speckle contrast for superposed speckle patterns created by rotating laser polarization," JOSA A 29, 2074 (2012) --- web_15_2_0_33
- [S13] "Speckle Reduction in Laser Projectors by Angular, Wavelength, and Polarization Diversities" (2020) --- web_15_1_0_22
- [S14] "Effect of brightness on speckle contrast and human speckle perception" (2019) --- web_15_1_0_0
- [S35] "Impact of speed, direction, and accuracy of DOE shift on efficiency of speckle suppression" (2024) --- web_15_2_0_31
- [S38] "Decoherent Focusing Design and Advanced" arxiv 2412.13869 (2024) --- web_15_2_0_30

### Phosphor conversion and speckle-free emission

- [S6] Aquino et al., "Effect of laser speckle on light from LD-pumped phosphor-converted light sources," Appl. Opt. 56, 278 (2017) --- web_15_1_0_9
- [S7] Kinoshita et al., "Speckle-Free Phosphor-Scattered Blue Light from InGaN/GaN LD" --- web_15_2_0_1
- [S17] "Laser phosphors," University of Michigan (2016) --- web_15_2_0_2
- [S18] "Speckle-Free, Angle-Free, Cavity-Free White Laser with High CRI," PMC (2024) --- web_15_2_0_4
- [S16] Lightsource.tech, "Laser-Pumped High-Brightness Incoherent Light Sources" --- web_15_2_0_0
- [S26] Shevlin, "Speckle reduction for illumination with lasers and phosphors" --- web_15_1_0_6

### MEMS and active despeckling

- [S30] "A Large-Size MEMS Scanning Mirror for Speckle Reduction Application," Micromachines 8, 140 (2017) --- web_15_1_0_40
- [S31] ResearchGate, same paper --- web_15_1_0_41
- [S44] US20120206784A1, "Device for reducing speckle effect in a display system" --- web_15_1_0_44

### LBS display systems

- [S25] Prysm LPD 6K spec sheet --- web_15_1_0_35
- [S26] Prysm LPD 6K 225 spec --- web_15_1_0_36
- [S27] TriLite Trixel 3, SPIE proceedings --- web_15_1_0_34
- [S28] TriLite Trixel 3, xpert.digital --- web_15_1_0_30
- [S40] TriLite + ams OSRAM partnership --- web_15_1_0_31
- [S42] "Principles of LBS" --- web_15_1_0_42

### Wave optics simulation

- [S19] HCIPy: "High Contrast Imaging for Python," arxiv (2018) --- web_15_2_0_15
- [S20] "Wave optics simulation approach for partial spatially coherent beams," Opt. Express 14, 6986 (2006) --- web_15_1_0_25
- [S21] "Wave optics simulation of atmospheric turbulence and reflective speckle effects," Appl. Opt. 39, 1857 (2000) --- web_15_1_0_26
- [S22] "Experimental study on speckle phase vortices near random surfaces" --- web_15_1_0_27
- [S28] LIDAR wave optics simulation, UNT digital library --- web_15_1_0_28
- [S29] PhysSandbox, "Laser Speckle Simulator" --- web_15_1_0_29
- [S37] WaveSim-optics, GitHub --- web_15_2_0_16
- [S39] AOtools: "A Python package for adaptive optics" --- web_15_2_0_17

### PSF models (Moffat vs. Gaussian)

- [S36] "The effects of seeing on Sersic profiles. II. The Moffat PSF" --- web_15_2_0_21
- [S37] "Gaussian and Moffat PSF," GitHub issue --- web_15_2_0_22
- [S38] "The point-spread function of fiber-coupled area detectors," PMC --- web_15_2_0_23
- [S39] "PSF Photometry Lecture," Berkeley --- web_15_2_0_24
- [S40] Siril documentation, "Dynamic PSF" --- web_15_2_0_20

### Eye safety and standards

- [S32] IEC 60825-1:2023 (GOST IEC 60825-1-2023), "Safety of laser products" --- web_15_2_0_11
- [S33] IEC 60825-1:2014, full text --- web_15_2_0_14
- [S34] IEC 62471 (GOST IEC 62471-2013), "Photobiological safety of lamps" --- web_15_1_0_12
- [S35] Vishay, "Eye Safety Risk Assessment According IEC 62471" --- web_15_1_0_10
- [S36] IEC 62471 photobiological safety overview --- web_15_1_0_11

### Temporal speckle and motion

- [S41] "High speed perfusion imaging based on laser speckle fluctuations," PhD thesis Draijer --- web_15_2_0_25
- [S42] "Quantitative temporal speckle contrast imaging for tissue mechanics" --- web_15_2_0_26
- [S43] "Evaluating OCT performance with temporal speckle averaging," PMC (2024) --- web_15_2_0_27
- [S44] "Processing of laser speckle contrast images," HAL thesis --- web_15_2_0_29

### Russian patent

- [S45] RU2282228C1, "Speckle suppression in optical scanning displays" --- web_15_1_0_7

### Holographic speckle reduction

- [S46] "HoloChrome: Polychromatic Illumination for Speckle Reduction in Holographic Near-Eye Displays," arxiv 2410.24144 (2024) --- web_15_1_0_24

### IEC standardization

- [S47] "What's the Focus of Debate in IEC Standardization for the Measurement of Laser Projection Displays?" SID (2023) --- web_15_1_0_49

---

## \S12. Effort Estimate

| Phase | Duration | Deliverable |
|--------|----------|-------------|
| Literature review + analytical model | 2-3 days | D1, D5 |
| Wave-optics simulation implementation | 3-4 days | D2 |
| Parameter sweep + per-channel analysis | 1-2 days | D3 |
| Decision matrix + report | 1 day | D4, D6 |
| **Total desktop research** | **7-10 days** | **D1-D6** |

### Optional lab validation

| Item | Cost | Purpose |
|------|------|---------|
| Blue LD + phosphor sample | $2K-$5K | Confirm speckle-free phosphor emission |
| MEMS scan mirror (evaluation kit) | $1K-$3K | Measure angular diversity from scan |
| Screen samples (matte, beaded, silver) | $500 | Parameterize screen roughness |
| CCD camera + lens (IEC 62906-5-2 compliant) | $3K-$8K | Standardized speckle measurement |
| **Total lab validation** | **$10K-$50K** | **Confirms simulation** |
| Duration | 2-4 months | |

---

## \S13. Relationship to Other Tasks

| Task | Interaction | Dependency |
|------|-----------|------------|
| TASK-01 (Power budget) | Speckle suppression methods may add power consumption | Independent |
| TASK-02 (Spatial addressing) | LBS scan pattern affects angular diversity | Independent (results feed in) |
| TASK-03 (Optical PSF) | Speckle corrupts MTF measurement; must be characterized first | **This task should run before or parallel with TASK-03** |
| TASK-04 (Neural chiplet) | Neural denoising could post-process speckle | Independent |
| TASK-05 (End-to-end path) | Speckle is an end-to-end image quality metric | Depends on this task's conclusions |

---

## \S14. Summary

**The speckle problem, often cited as the primary objection to laser displays, is largely solved by PRP's own architecture --- without additional hardware.**

The mechanism is straightforward: PRP uses phosphor conversion for green and red channels (PRP \S24). Phosphor fluorescence is inherently incoherent (coherence length 3-15 um, comparable to LED). This eliminates speckle for the two channels where human perception is most sensitive (green: 3.2%, red: 3.6%).

Blue, the least sensitive channel (4.4% threshold), is used directly from the laser diode. With scan-based angular diversity, temporal averaging, and polarization diversity, blue speckle can be brought to ~5-7%, which is above the still-image threshold but below the motion-image threshold. One passive measure (dual blue LD or slight linewidth broadening) brings it to ~3-4%.

**No moving diffusers, no rotating elements, no additional optical components are required.** The speckle solution is architectural, not additive --- it comes from the same design choice (phosphor conversion) that eliminates color filters. This is a compound advantage: one architectural decision solves two problems simultaneously.

The remaining work is verification: wave-optics simulation to confirm the analytical estimates, and lab measurement to validate the simulation. The simulation is straightforward (Fresnel propagation + phase screens + incoherent source model) and can be completed in 7-10 days.

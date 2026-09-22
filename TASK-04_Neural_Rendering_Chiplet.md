# TASK-04: Neural Rendering Chiplet — Feasibility & Characterisation

> **Stage 4 of the PRP Experimental Roadmap (§28)**
> Companion document to *Photonic Rendering Pipeline v3*.
> License: CC BY 4.0

---

## §1. Research Question

**Can a photonic neural network chiplet perform a controlled rendering-relevant workload (image reconstruction, super-resolution, or denoising) at energy, latency, precision, and scale sufficient to serve as a module in the PRP pipeline?**

This is **not** asking whether a photonic neural network can replace a GPU for full 4K rendering. It asks whether a *bounded, well-defined* workload — analogous to what ACCEL does for classification, but for pixel generation / refinement — can run on a photonic chiplet with acceptable metrics.

### Falsification criterion

If photonic neural networks cannot achieve **≥8-bit effective precision**, **<100 ns latency** for a small-image workload, and **<100 fJ/MAC** energy simultaneously at the scale of at least 64×64 pixels — then the chiplet concept is not viable for the PRP pipeline, and Stage 5 (direct photonic rendering-to-display) cannot rely on a photonic neural pre-processor.

---

## §2. Scope

Stage 4 introduces a **photonic neural network chiplet** for a controlled workload within the PRP architecture. The chiplet is not the full renderer — it is a *module* that performs a specific, measurable task:

- **Image reconstruction** (deconvolution, inverse filtering)
- **Super-resolution** (upscaling low-res input to higher-res output)
- **Denoising** (removing sensor/scanner noise)
- **Frame interpolation** (generating intermediate frames)

The chiplet sits between the electronic control domain and the optical output domain (PRP §28, Stage 4 diagram):

```
Electronic control → Photonic neural rendering → Optical image field → Optical interconnect → Optical display
```

### What is measured

Per the PRP roadmap, seven metrics must be characterised:

1. Energy per operation
2. Latency
3. Optical loss
4. Precision (effective bit depth)
5. Error rate
6. Thermal stability
7. Scaling behaviour

---

## §3. Background: State of the Art

### 3.1 ACCEL — All-analog Photoelectronic Chip

ACCEL [1] (Tsinghua, *Nature* 2023) is the closest existing analogue to what Stage 4 envisions — a photonic chip that processes image data end-to-end.

| Metric | ACCEL value | Reference |
|--------|------------|-----------|
| Latency per frame | 72 ns (3-class), 240 ns (10-class) | [1] |
| Energy per frame | 4.38 nJ | [1] |
| Operations per frame | 3.28 × 10⁸ (3-class ImageNet) | [1] |
| Implied energy per MAC | ~13.4 fJ/MAC | calculated |
| Accuracy (Fashion-MNIST) | 85.5% | [1] |
| Accuracy (3-class ImageNet) | 82.0% | [1] |
| Accuracy (10-class MNIST) | 97.1% | [1] |
| Image size | 28×28 (MNIST), 400×400 (OAC layers) | [1] |
| Comparison: NVIDIA A100 | 0.26 ms/frame, 18.5 mJ/frame | [1] |

ACCEL uses diffractive optical computing as an optical encoder (OAC) for feature extraction, followed by an electronic analog computing chip (EAC) — **no ADC between stages**. The 72 ns latency includes three optical pulses for 3-class classification.

**Relevance to PRP**: ACCEL demonstrates that photonic pre-processing of image data at nanosecond latency is real. However, it performs *classification*, not *generation* — it reduces an image to a label, not to another image. Stage 4 needs the inverse: image → image.

### 3.2 OPCA — Optical Parallel Computational Array

OPCA [2] (Tsinghua, *Optica* 2024) extends the concept to end-to-end optical image processing, transmission, and reconstruction.

| Metric | OPCA value | Reference |
|--------|-----------|-----------|
| Response time | 6 ns | [2] |
| Optical bandwidth | 160 nm (1480–1640 nm) | [2] |
| Processing bandwidth | ~100 billion pixels | [2][3] |
| Array size | 4×4 (demonstrated) | [2] |
| Accuracy (MNIST) | 73.5% | [4] |
| Reconstruction PSNR | 11.32–11.69 dB | [2] |
| Architecture | Ring resonator array, WDM | [2] |

OPCA performs both summation and subtraction (constructive/destructive interference) and can transmit results via optical fibre without optoelectronic conversion.

**Relevance to PRP**: OPCA is the *only* demonstrated chip that performs image reconstruction (not just classification) in the optical domain. The 11.3 dB PSNR is low by display standards (~20 dB would be acceptable), but the architecture is extensible. The 4×4 array is far from 4K, but the 6 ns response time is the right order.

### 3.3 PDNN — Photonic Deep Neural Network

PDNN [5] (MIT, *Nature* 2022) demonstrates on-chip deep neural network for sub-nanosecond image classification.

| Metric | PDNN value | Reference |
|--------|-----------|-----------|
| Classification time | <570 ps | [5] |
| Accuracy (2-class) | >93.8% | [5] |
| Accuracy (4-class) | >89.8% | [5] |
| Architecture | On-chip pixel array, cascaded neurons | [5] |
| Nonlinearity | Opto-electronic (O/E/O) | [5] |
| Supply light | 1559.93 nm laser, uniform distribution | [5] |

**Relevance to PRP**: PDNN demonstrates scalability to deep networks (multiple layers) with uniform optical supply — a key architectural requirement for a rendering chiplet that needs consistent per-neuron output range.

### 3.4 FICONN — Single-chip Photonic Deep Neural Network

FICONN [6] (*Nature* 2024 preprint) demonstrates a fully integrated coherent optical neural network.

| Metric | FICONN value | Reference |
|--------|-------------|-----------|
| Latency | 410 ps | [6] |
| Architecture | 6 neurons, 3 layers | [6] |
| Nonlinearity | All-optical (DFB-SA array) | [6] |
| Accuracy (6-class vowel) | 92.5% | [6] |
| Energy efficiency (MZI mesh) | 1.39 TOPS/W | [6] |
| Energy efficiency (DFB-SA) | 987.65 GOPS/W | [6] |
| Training | Forward-only (backpropagation-free) | [6] |

**Relevance to PRP**: FICONN is the first chip with *all-optical* nonlinear activation (no O/E/O between layers), which is critical for a rendering pipeline that must stay in the optical domain. 410 ps latency per inference is well within the PRP latency budget.

### 3.5 SLiM — Hundred-layer Photonic Deep Learning

SLiM [7] (Tsinghua, *Nature Communications* 2025) demonstrates 100-layer photonic networks with error tolerance.

| Metric | SLiM value | Reference |
|--------|-----------|-----------|
| Max depth | 100 layers | [7] |
| Error std (signal intensity) | 6–20% | [7] |
| Accuracy (18-layer, 1 GHz) | 80.9% | [7] |
| Accuracy (100-layer, 10 GHz) | 85.4% | [7] |
| Parameters | up to 345 million | [7] |
| Error tolerance | std up to 44% input error | [7] |
| Temporal stability | <1.0% mean error over 10 h | [7] |

**Relevance to PRP**: SLiM shows that photonic networks can scale to 100 layers with error-tolerant architectures — the error rate of 9–10% at 1 GHz is high but stabilised by the SLiM error-bounding criterion. For rendering, where output is an image (not a label), 10% error per element would be visible; the question is whether error-tolerant training can reduce this to <1% for pixel-level tasks.

### 3.6 HOP — Hybrid Optical Processor (16-bit precision)

HOP [8] (arxiv 2024) demonstrates digital-analog hybrid matrix multiplication with 16-bit precision.

| Metric | HOP value | Reference |
|--------|----------|-----------|
| Calculation precision | 16 bits | [8] |
| Pixel error rate (16-bit HDIP) | 1.8 × 10⁻³ at 18.2 dB SNR | [8] |
| Pixel error rate (8-bit, 35 dB OSNR) | 6.0 × 10⁻⁵ | [8] |
| Analog-only precision | 3.6 bits (σ = 0.027) | [8] |
| Architecture | Digital-analog hybrid | [8] |

**Relevance to PRP**: HOP bridges the precision gap. Pure analog photonic computing gives ~3–4 bits; the hybrid approach achieves 16 bits for image processing with acceptable error rates. For PRP, 10–12 bits per channel are needed for HDR — HOP's hybrid scheme is a candidate architecture.

---

## §4. The Seven Metrics — Data & Analysis

### 4.1 Energy per Operation

| Platform | Energy per MAC | Notes | Source |
|----------|---------------|-------|--------|
| ACCEL | ~13.4 fJ/MAC (calculated) | 4.38 nJ / 3.28×10⁸ ops | [1] |
| Photonic MZI mesh (theoretical) | 4 aJ/MAC (4-bit), 1 fJ/MAC (8-bit) | Shot-noise limited | [9] |
| Photonic (6G accelerator) | 0.18 pJ/MAC | 71× better than GPU (12.8 pJ) | [10] |
| Memresonator (switching) | 0.15 pJ (SET), 0.36 pJ (RESET) | Non-volatile, zero static | [11] |
| Electronic (5 nm CMOS) | ~1 pJ/MAC | Lower bound | [10] |
| GPU (NVIDIA A100) | 12.8 pJ/MAC | For comparison | [10] |
| Electronic ASIC (7 nm) | 2.3 pJ/MAC | Specialised | [10] |
| FICONN (MZI mesh) | 1.39 TOPS/W → 719 pJ/MAC | Includes thermal overhead | [6] |
| Reported photonic (50–100 fJ) | 50–100 fJ/MAC | Under specific conditions | [10] |

**Analysis**: The theoretical floor for photonic MAC is ~4 aJ for 4-bit operations [9], but real devices include thermal control (15% of energy budget [10]), photodetection + ADC (45% [10]), and optical modulation (30% [10]). Practical energy is 50 fJ – 1 pJ/MAC depending on architecture. ACCEL's 13.4 fJ/MAC is the best demonstrated for image processing.

**For PRP**: A 4K60 frame has ~8.3 million pixels × 3 channels = 25 million values. If the chiplet performs 10 MAC per pixel (lightweight reconstruction), that's 250 million MAC/frame. At 50 fJ/MAC, that's 12.5 μJ/frame — negligible. At 1 pJ/MAC, 250 μJ/frame — still negligible compared to a GPU's ~200 mJ/frame. Energy is **not the bottleneck**.

### 4.2 Latency

| Platform | Latency | Workload | Source |
|----------|---------|----------|--------|
| PDNN | <570 ps | 2-class classification | [5] |
| FICONN | 410 ps | 6-class vowel | [6] |
| OPCA | 6 ns | 4×4 image processing | [2] |
| ACCEL | 72 ns | 400×400, 3-class | [1] |
| 6G photonic accelerator | 850 ns (end-to-end) | 256-antenna beamforming | [10] |
| GPU (A100) | 260 μs | Same task as ACCEL | [1] |

**Analysis**: Photonic neural networks operate at picosecond-to-nanosecond latency for small workloads. The PRP latency budget is 16.67 ms (60 fps) for the entire pipeline; the chiplet alone should use <1 ms.

At 6 ns per inference (OPCA), even 100 sequential inferences for a 4K frame (tiled 64×64 blocks) would be 600 ns — well within budget. The bottleneck is not compute latency but **scaling** (see §4.7).

**For PRP**: Latency is **not the bottleneck** at the chiplet level. The concern is whether the *scaling path* from 4×4 to 3840×2160 preserves the per-element latency.

### 4.3 Optical Loss

| Component | Loss | Notes | Source |
|-----------|------|-------|--------|
| Si waveguide propagation | 0.026 dB/cm (best) | Research-grade | [12] |
| Si waveguide propagation | 0.5–3 dB/cm (typical) | Foundry PDK | [13] |
| SiN waveguide | <0.5 dB/cm | Low-loss platform | [13] |
| MZI (thermo-optic) | 0.5–1 dB per device | Insertion loss | [14] |
| MZI mesh (128×128, MZI-based) | 64–128 dB | Cascade of 128 devices | [14] |
| MOMZI PTC (128×128) | 0.25 dB lower | Near-constant IL | [14] |
| Coupler (off-chip → on-chip) | 1–3 dB per facet | Grating/edge coupler | [12] |
| Photodetector | 0.5–1 dB | Insertion | [12] |

**Analysis**: Optical loss is the **most critical scalability bottleneck**. A 128×128 MZI mesh loses 64–128 dB through cascaded devices [14] — this is catastrophic. The MOMZI multi-operand approach [14] reduces this to near-constant insertion loss (~0.25 dB) by placing modulators in parallel rather than series.

For a rendering chiplet processing 64×64 tiles, a 64×64 MZI mesh would cascade ~64 devices → 32–64 dB loss. This is still high and requires optical amplification or a different architecture (MOMZI, MRR bank, or diffractive).

**For PRP**: Optical loss **is a bottleneck** for large meshes. The architecture must use parallel-modulator designs (MOMZI [14]), MRR weight banks, or diffractive optics to avoid cascaded loss. The PRP chiplet should target <10 dB total on-chip loss.

### 4.4 Precision (Effective Bit Depth)

| Architecture | Effective bits | Notes | Source |
|-------------|---------------|-------|--------|
| MZI mesh (phase encoding) | 5–9 bits | With thermal stabilisation | [15] |
| MRR weight bank | 5.1 bits | Measured weight accuracy | [16] |
| Analog photonic (general) | 3–5 bits | Inherent noise floor | [15] |
| HOP (hybrid digital-analog) | 16 bits | For image processing | [8] |
| Photonic (with dithering) | 9 bits | Effective attenuation | [15] |
| BPNC (butterfly, 3-bit DAC) | 3 bits (weight), 94.16% acc | MNIST | [17] |
| Shot-noise-limited (4-bit) | 4 bits | Theoretical floor | [9] |
| Shot-noise-limited (8-bit) | 8 bits | Requires 232 photons/sample | [9] |

**Analysis**: Pure analog photonic computing is limited to 3–5 bits by noise (shot noise, thermal drift, fabrication variation) [9][15]. This is insufficient for rendering (10–12 bits needed for HDR). Hybrid digital-analog approaches like HOP [8] achieve 16 bits by encoding in binary words and processing bit-slices optically, but at the cost of serial processing (multiple passes).

For 8-bit precision at 1550 nm, 232 photons per sample are needed, giving ~550 pJ per sample and >1 pJ/MAC [9] — this erases the photonic energy advantage.

**For PRP**: Precision **is a bottleneck** for pure analog approaches. The chiplet must either:
1. Use hybrid encoding (HOP-style, 10–16 bits, multiple passes)
2. Accept 4–8 bits and rely on temporal dithering / error diffusion (trading latency for precision)
3. Use a photonic front-end for coarse computation + electronic refinement

### 4.5 Error Rate

| Platform | Error metric | Value | Source |
|----------|-------------|-------|--------|
| SLiM (100-layer, 1 GHz) | Accuracy degradation | 0.48% from ideal | [7] |
| SLiM (standard deviation) | Signal error std | 4.5–6.2% | [7] |
| SLiM (error rate) | At 1/5/10 GHz | 9.04/9.22/10.34% | [7] |
| HOP (16-bit) | Pixel error rate | 1.8 × 10⁻³ | [8] |
| HOP (8-bit, 35 dB) | Pixel error rate | 6.0 × 10⁻⁵ | [8] |
| FICONN | Accuracy | 92.5% (6-class) | [6] |
| Shen et al. (MZI ONN) | Accuracy | 76.7% (best calibrated) | [18] |
| Error-aware training | Robust to 5× fabrication error | Static hardware errors | [19] |
| OPCA (MNIST) | Accuracy | 73.5% | [4] |

**Analysis**: Error rates of 9–10% per element are typical for deep photonic networks at GHz speeds [7]. For classification, this is tolerable (networks are robust to label-level errors). For rendering, 10% per-pixel error is visible noise.

The error sources are: (1) fabrication variation, (2) thermal crosstalk, (3) shot noise, (4) photodetection noise, (5) finite encoding precision [18].

Error-aware training [19] can make networks robust to 5× the current fabrication error, but this addresses *systematic* errors, not random per-pixel noise.

**For PRP**: Error rate **is a bottleneck** for pixel-level tasks. Strategies:
1. Error-aware training with pixel-level loss (not just classification loss)
2. Temporal averaging (multiple passes, frame accumulation)
3. Hybrid: photonic coarse pass + electronic fine correction
4. Accept higher error for non-critical applications (e.g., preview, not final frame)

### 4.6 Thermal Stability

| Issue | Impact | Magnitude | Source |
|-------|--------|-----------|--------|
| MRR resonance shift | 0.05 K → significant wavelength shift | ~0.05 K sensitivity | [20] |
| Thermal crosstalk (adjacent rings) | 50% at 100 μm proximity | 2D heat diffusion | [21] |
| MZI bias drift | Time-dependent phase error | All materials (LiNbO₃, InP, Si) | [22] |
| Environmental temp drift | Weight error NMAE 0.065–0.311 | Linear 1 K drift | [20] |
| DOCTOR remediation | Real-time calibration | Recovers accuracy after drift | [20] |
| SLiM temporal stability | <1.0% error over 10 h | 8-dimensional output | [7] |
| Thermal control energy | 15% of total energy budget | Phase shifters, heaters | [10] |
| Memresonator | Zero static power (non-volatile) | 0.15 pJ SET, retains state | [11] |

**Analysis**: Thermal stability is a **known, actively researched problem**. Silicon's thermo-optic coefficient is high (~1.86 × 10⁻⁴ K⁻¹), making MRRs sensitive to <0.1 K changes [20]. Crosstalk between adjacent heaters affects 50% at 100 μm [21].

Solutions being developed:
- DOCTOR [20]: dynamic on-chip calibration, training-free, data-free
- Thermal isolation trenches [18]
- Non-volatile memresonators [11]: zero static power, state retained
- Feedback control with dithering [15][22]

**For PRP**: Thermal stability **is a manageable bottleneck**. The chiplet will require:
1. Active thermal stabilisation (feedback loops, <0.01 K control)
2. Thermal isolation between neighbouring modulators
3. Non-volatile weight storage (memresonator) to eliminate static thermal load
4. Periodic recalibration (like DOCTOR [20])

This adds ~15% to energy budget [10] and requires calibration electronics, but it does not break the architecture.

### 4.7 Scaling Behaviour

| Metric | Current scale | 4K requirement | Gap | Source |
|--------|--------------|----------------|-----|--------|
| Input dimension (PDNN) | ~10 pixels | 8.3M pixels | 6 orders | [5] |
| Input dimension (OPCA) | 4×4 = 16 | 3840×2160 | 5 orders | [2] |
| Input dimension (SLiM) | 64 | 8.3M | 5 orders | [7] |
| MZI mesh size | 64×64 (demonstrated) | Would need O(N²) = 69 billion MZIs | Infeasible | [14][23] |
| PDONN depth | 4 layers | Unknown (task-dependent) | Unknown | [24] |
| Parameters | 345M (SLiM) | ~100M–1B (lightweight SR) | Feasible | [7] |
| Optical loss (128×128 MZI) | 64–128 dB | Must be <10 dB | 50–120 dB gap | [14] |
| MZI count (N×N unitary) | N² | 69 trillion for 8.3M | Infeasible on single chip | [23] |

**Analysis**: Scaling is the **fundamental bottleneck**. A direct MZI mesh for 4K is infeasible — O(N²) scaling means 69 trillion MZIs for 8.3M inputs [23]. Even 128×128 requires 128 cascaded MZIs per optical path, losing 64–128 dB [14].

The path forward is **tiling + tiling-aware architecture**:
1. **Spatial tiling**: Process 64×64 or 128×128 tiles, scan across the frame
2. **Wavelength multiplexing**: OPCA uses WDM for parallel channels [2]
3. **Tensor decomposition**: Reduce MZI count by 42.7× via tensor-train compression [25]
4. **Pruning**: Reduce MZI count by 50–70% with <0.2% accuracy loss [17][26]
5. **Diffractive optics**: Subwavelength structures scale without O(N²) MZIs [27]
6. **Multi-operand devices**: MOMZI reduces cascaded loss to near-constant [14]

**For PRP**: Scaling **is the primary risk** for Stage 4. The chiplet must NOT attempt full-frame processing. It must process tiles (64×64 or 128×128) with a photonic core, scanning across the frame. This is architecturally compatible with LBS scanning (TASK-02) — the chiplet processes one tile while the scanner addresses the next.

---

## §5. Gap Analysis: Classification vs. Rendering

All demonstrated photonic neural networks perform **classification** (image → label) or **sensing** (image → feature). Stage 4 needs **generation** (image → image) or **reconstruction** (degraded image → clean image). The gap:

| Aspect | Classification (demonstrated) | Rendering (needed) | Gap |
|--------|------------------------------|-------------------|-----|
| Output dimension | 1–10 (label) | 25M (4K RGB) | 7 orders |
| Precision needed | 1 bit (correct/incorrect) | 8–12 bits per channel | 8–12× |
| Error tolerance | ~10% (classification robust) | <1% (visible noise) | 10× |
| Loss function | Cross-entropy | MSE / perceptual | Different |
| Training data | Labelled images | Paired (degraded, clean) | Different |
| Output type | Discrete (class) | Continuous (pixel values) | Fundamental |

**This gap is the core research question of TASK-04.** It is not addressed by any existing photonic neural network demonstration. However, the *building blocks* exist:

- OPCA [2] performs reconstruction (11.3 dB PSNR) — the only demonstrated image-to-image photonic chip
- HOP [8] performs 16-bit convolution for image processing — the only high-precision photonic image processor
- SLiM [7] tolerates 44% input error — error-tolerant training exists
- SBS activation [28] provides all-optical nonlinearity with 20 dB net gain — deep network building block

---

## §6. Deliverables

### D1: Architecture Proposal for a Photonic Rendering Chiplet

Define a concrete architecture for a tile-based photonic rendering chiplet, specifying:
- Tile size (64×64, 128×128, or other)
- Photonic core type (MZI mesh, MRR bank, diffractive, hybrid)
- Precision strategy (analog 4–8 bit, hybrid 10–16 bit)
- Nonlinearity (O/E/O, all-optical SBS, or optoelectronic)
- Weight storage (volatile thermal, non-volatile memresonator)
- Optical loss budget (target <10 dB on-chip)
- Tiling strategy (serial scan, parallel wavelength, or hybrid)

### D2: Workload Definition

Define 2–3 concrete rendering-relevant workloads suitable for a chiplet:
- **W1: 2× super-resolution** (64×64 → 128×128, 3-channel, 8-bit)
- **W2: Denoising** (64×64 input, Gaussian σ=0.05, 3-channel, 8-bit)
- **W3: Deconvolution** (64×64 input, known PSF, 3-channel, 8-bit)

For each: input format, output format, acceptable PSNR (≥20 dB), acceptable SSIM (≥0.95), acceptable latency (<1 ms per tile).

### D3: Simulation Framework

Build a Python simulation that models:
- Photonic core forward pass (with noise: shot noise, thermal drift, fabrication variation)
- Precision quantisation (4, 8, 12, 16 bit modes)
- Optical loss accumulation
- Tiling scan over a full 4K frame
- Metric extraction: PSNR, SSIM, latency, energy per frame

### D4: Benchmark Against Electronic Baselines

Compare the simulated photonic chiplet against:
- GPU (same workload, same tile size)
- Electronic ASIC (dedicated SR/denoise chip)
- Ideal (no noise, full precision)

### D5: Decision Matrix

| Outcome | Criterion | Action |
|---------|-----------|--------|
| **Supported** | ≥8-bit precision, ≥20 dB PSNR, <1 ms/tile, <100 fJ/MAC | Proceed to Stage 5 |
| **Partial** | 4–8 bit precision, ≥15 dB PSNR, <10 ms/tile | Redefine workload, add electronic refinement |
| **Falsified** | <4-bit precision or <10 dB PSNR or >100 ms/tile | Photonic chiplet not viable; use electronic renderer |
| **Precision-limited** | ≥20 dB PSNR but <8-bit | Investigate temporal dithering / error diffusion |
| **Scale-limited** | Works at 64×64 but not 128×128 | Investigate diffractive / MOMZI architecture |

### D6: Open Questions for Stage 5

If the chiplet is viable, what does Stage 5 (direct photonic rendering-to-display) require?
- Interface between chiplet output and optical display (free-space? fibre? waveguide?)
- Frame buffer elimination: can the chiplet stream directly to the scanner?
- Synchronisation with LBS / SLM addressing (TASK-02)
- HDR: can 10–12 bit be achieved through dithering at 120 Hz?

---

## §7. Relationship to Other PRP Sections

| PRP Section | Relationship |
|------------|-------------|
| §7 (Chiplet architecture) | TASK-04 defines the neural chiplet |
| §11 (Optical PSF reconstruction) | Chiplet could perform deconvolution complementing optical PSF (TASK-03) |
| §28 (Stage 4) | This task IS Stage 4 |
| §28 (Stage 5) | D6 feeds directly into Stage 5 planning |
| §31.3 (Precision/HDR) | §4.4 of this task addresses the precision question |
| §31.5 (Energy budget) | §4.1 of this task addresses energy |
| §46.5 (Machine vision) | Photonic neural processing is directly applicable to machine vision |

---

## §8. Effort Estimate

| Phase | Duration | Deliverable |
|-------|----------|------------|
| Literature synthesis & architecture design | 3–5 days | D1 |
| Workload definition | 1–2 days | D2 |
| Simulation framework | 5–7 days | D3 |
| Benchmarking | 2–3 days | D4 |
| Decision matrix & report | 1–2 days | D5, D6 |
| **Total (desktop)** | **12–19 days** | All |

### Optional laboratory validation (Stage 4b)

| Phase | Duration | Cost estimate |
|-------|----------|---------------|
| Chiplet fabrication (MPW run) | 4–6 months | $10K–$50K |
| Characterisation setup | 2–4 weeks | $5K–$20K (equipment) |
| Measurement campaign | 2–4 weeks | — |
| **Total (lab)** | **6–8 months** | **$15K–$70K** |

---

## §9. Sources

### Photonic neural network demonstrations

- [1] ACCEL — Nature 2023: https://www.nature.com/articles/s41586-023-06558-8
- [2] OPCA — ResearchGate (Optica 2024): https://www.researchgate.net/publication/380278953
- [3] OPCA coverage — TechTimes: https://www.techtimes.com/articles/305663/20240613/
- [4] OPCA — Academia.edu: https://www.academia.edu/143315081
- [5] PDNN — Nature 2022: https://www.nature.com/articles/s41586-022-04714-0
- [5a] PDNN — arXiv: https://arxiv.org/abs/2106.11747
- [6] FICONN — ResearchGate 2024: https://www.researchgate.net/publication/386341124
- [7] SLiM — Nature Communications 2025: https://www.nature.com/articles/s41467-025-65356-0
- [8] HOP — arXiv 2024: https://arxiv.org/html/2401.15061

### Energy and efficiency

- [9] Photonic MAC operations — SciSpace: https://scispace.com/pdf/photonic-multiply-accumulate-operations-for-neural-networks-1s3red5xlc.pdf
- [10] 6G photonic accelerator — IIETA: https://iieta.org/journals/ts/paper/10.18280/ts.430103
- [11] Memresonator — PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC10791609/

### Optical loss and scaling

- [12] Silicon photonic accelerators review — arXiv: https://arxiv.org/pdf/2101.01751
- [13] Optical filter design — UPM thesis: https://oa.upm.es/55951/1/TFM_AITOR_LOPEZ_HERNANDEZ.pdf
- [14] MOMZI multi-operand neurons — PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC11501373/
- [23] MZI scaling — UCLA: https://innovate.ee.ucla.edu/wp-content/uploads/2022/05/aop-14-2-209-compressed.pdf

### Precision and noise

- [15] Photonic analog signal processing — Chinese Optics Letters: https://www.opticsjournal.net/Articles/OJa745b786fce715df/FullText
- [16] Machine learning with neuromorphic photonics — Queen's University: https://www.queensu.ca/physics/shastrilab/sites/shastwww/files/uploaded_files/publications/journals/47_Ferreira_de_Lima-JLT-machine_learning_2019.pdf
- [17] BPNC butterfly neural chip — ACS Photonics: https://sites.utexas.edu/chen-server/wp-content/uploads/sites/5483/2022/11/acsphotonics.2c01188.pdf

### Thermal stability

- [18] Shen et al. — coherent nanophotonic circuits: https://eclass.uoa.gr/modules/document/file.php/DI356/shen2017.pdf
- [19] Noise-resilient photonic analog NN — IEEE JLT: https://colab.ws/articles/10.1109%2Fjlt.2024.3433454
- [20] DOCTOR — arXiv 2024: https://arxiv.org/pdf/2403.02688v1
- [21] Electronic-photonic thermo-optic tuning — JOS: https://www.jos.ac.cn/article/doi/10.1088/1674-4926/42/2/023104
- [22] Silicon photonics codesign — Columbia: https://lightwave.ee.columbia.edu/sites/lightwave.ee.columbia.edu/files/content/publications/2020/Silicon%20Photonics%20Codesign.pdf

### Scaling approaches

- [24] PDONN scaling up — Light: Science & Applications: https://www.nature.com/articles/s41377-025-02029-z
- [25] Scalable BP-free training of optical PINNs — arXiv: https://arxiv.org/pdf/2502.12384
- [26] Pruning optimisation of ONN — Frontiers: https://www.frontiersin.org/journals/advanced-optical-technologies/articles/10.3389/aot.2024.1501208/full
- [27] On-chip diffractive tensor processing — Photonics Research: https://www.opticsjournal.net/Articles/OJd02da4d2d1d8361f/FullText

### Activation functions

- [28] SBS all-optical activation — PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC12338876/
- [28a] SBS activation — ScienceDaily: https://www.sciencedaily.com/releases/2025/04/250414124725.htm
- [29] Review of nonlinear activation in ONNs — Researching.cn: https://www.researching.cn/ArticlePdf/m00090/2025/7/6/064004.pdf
- [30] Photonic NPU technology overview: https://www.photonicnpu.com/technology.html

### Reviews

- [31] Photonic neural networks review — ResearchGate: https://www.researchgate.net/publication/384193106
- [32] Photonic DL accelerators review — Frontiers: https://www.frontiersin.org/journals/physics/articles/10.3389/fphy.2024.1369099/full
- [33] Exploring types of PNNs — PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC11054149/
- [34] Photonic computing overview — AI Advances: https://ai.gopubby.com/ai-at-the-speed-of-light-how-photonic-neural-networks-could-transform-computing-5aacbef8f536
- [35] Photonic neural network market — DataIntelo: https://dataintelo.com/report/silicon-photonic-optical-neural-network-chip-market
- [36] Silicon photonic NN accelerators — Colorado State: https://www.engr.colostate.edu/~mnikdast/files/papers/Mahdi_C49.pdf
- [37] Research progress in ONNs — PhotoniX/Springer: https://link.springer.com/article/10.1186/s43074-021-00026-0
- [38] Deep learning for photonic design — Frontiers: https://www.frontiersin.org/journals/materials/articles/10.3389/fmats.2021.791296/full
- [39] Deep learning for photonic inverse design — Encyclopedia MDPI: https://encyclopedia.pub/entry/47251
- [40] Optical neural networks progress and challenges — ResearchGate: https://www.researchgate.net/publication/384193106

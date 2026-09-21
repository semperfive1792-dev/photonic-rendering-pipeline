# Photonic Rendering Pipeline

**A Hybrid Electronic–Photonic Architecture for Direct Optical Image Formation**

Author: Semperfive  
Date: September 2026  
License: Creative Commons Attribution 4.0 International (CC BY 4.0)  
Publication status: Architectural concept / research proposal

## Abstract

Modern graphics systems increasingly compensate for the limitations of their underlying architecture by adding additional processing stages.

Higher image quality introduces additional reconstruction algorithms.  
Higher resolution introduces wider interfaces and compression.  
Variable refresh introduces synchronization protocols.  
HDR introduces additional processing and display-side control.  
Neural rendering introduces increasingly complex data movement between compute units, memory and display pipelines.

This document proposes a different architectural approach.

Instead of continuously optimizing the conventional digital display pipeline, the proposed architecture removes unnecessary representations between rendering and light emission.

The central idea is to divide the graphics system into two domains:

1. **Electronic domain** — general-purpose computation, control, memory, simulation, operating-system interaction, scheduling and other tasks for which conventional electronics remain appropriate.
2. **Photonic domain** — neural rendering, selected image transformations, high-bandwidth signal distribution and optical image formation.

The system therefore does not attempt to build a fully photonic computer.

Instead, a conventional electronic processor is combined with one or more photonic chiplets. Neural rendering and image-domain operations that naturally map to optical computation are performed in the photonic domain. The resulting optical representation is then transmitted directly to an optical display subsystem without first being converted back into a conventional digital framebuffer and serialized through a traditional display interface.

Conceptually:

```
Application / Engine
        |
        v
Electronic CPU / GPU / NPU
        |
        | scene data, control, geometry,
        | simulation, memory, scheduling
        v
Photonic Rendering Chiplet
        |
        | neural rendering
        | reconstruction
        | optical image transformation
        v
Optical Image Representation
        |
        | waveguide / fiber / photonic interconnect
        v
Optical Output Module
        |
        | spatial optical formation
        v
Laser / Phosphor / Projection System
        |
        v
Human visual system
```

The proposal is therefore not based on the assumption that every component must become optical.

Its central principle is simpler:

> Keep computation electronic where electronics are appropriate. Keep image data optical where the final information is fundamentally optical.

The potential benefits include reduced data movement, elimination of several intermediate representations, lower interface complexity, direct optical broadcast, reduced dependence on rasterized display protocols, and a fundamentally different approach to image reconstruction.

The numerical performance figures discussed in this document are potential targets or architectural envelopes, not claimed experimental results.

## 1. Core Thesis

The fundamental observation behind this architecture is that a display ultimately produces light.

Nevertheless, conventional graphics systems often perform a long sequence of conversions before reaching that final physical representation:

```
Scene
  -> Electronic computation
  -> Digital framebuffer
  -> Digital image processing
  -> Digital serialization
  -> Digital transmission
  -> Digital reconstruction
  -> Panel timing
  -> Pixel driving
  -> Optical emission
```

The proposal asks a simple question:

> Which of these intermediate representations are actually necessary?

If a portion of rendering is already being performed by an analog or photonic computational system, and the final destination is an optical field, then repeatedly converting the information between electronic and optical representations may be unnecessary.

The proposed architecture therefore attempts to preserve the optical representation for as much of the final image pipeline as practical.

This leads to a general architectural principle:

> Do not optimize an unnecessary representation. Remove it.

## 2. Design Philosophy

The proposal is based on five principles.

### 2.1 Heterogeneity instead of technological purity

The system should not attempt to replace electronics wholesale.

Electronics remain the preferred medium for:

- operating-system execution;
- application logic;
- scene management;
- physics simulation;
- control flow;
- branching;
- memory management;
- scheduling;
- general-purpose computation;
- device management;
- error handling;
- configuration;
- system control.

Photonic hardware is introduced only where its properties provide a useful architectural advantage.

### 2.2 Compute where the mathematics is naturally suited to optics

Many neural-network operations consist primarily of large linear transformations, weighted combinations and massively parallel signal propagation.

These are natural candidates for photonic computation.

The purpose is therefore not:

> Make the GPU photonic.

The purpose is:

> Move selected computational workloads into a photonic accelerator when doing so reduces energy, latency or data movement.

### 2.3 Preserve optical data as optical data

The conventional architecture frequently follows:

```
optical -> electronic -> digital -> electronic -> optical
```

The proposed architecture attempts to replace this with:

```
electronic -> photonic -> optical
```

where appropriate.

This is particularly relevant when the output of a photonic neural renderer can remain in an optical representation until it reaches the display subsystem.

### 2.4 Separate architecture from implementation technology

The concept does not depend on one particular implementation.

The photonic processing element could use different technologies, including:

- Mach–Zehnder interferometer networks;
- microring-based architectures;
- wavelength-division multiplexing;
- free-space optical computing;
- diffractive optical elements;
- photonic crystal or waveguide structures;
- integrated laser sources;
- hybrid silicon photonics;
- heterogeneous photonic/electronic integration.

Likewise, the optical output stage could use:

- laser-phosphor systems;
- direct laser projection;
- scanning optical systems;
- spatial optical emitters;
- free-space optical coupling;
- other emerging optical display technologies.

The architecture is therefore intended to remain technology-agnostic at the component level.

## 3. Proposed System Architecture

The complete system can be divided into six functional domains.

```
+----------------------------------------------------------+
|                 ELECTRONIC COMPUTE DOMAIN                |
|                                                          |
|  CPU / GPU / NPU / Memory / Control / Simulation         |
+----------------------------+-----------------------------+
                             |
                             | scene representation,
                             | neural inputs, parameters
                             v
+----------------------------------------------------------+
|                  PHOTONIC CHIPLET                        |
|                                                          |
|  Neural rendering                                        |
|  Neural reconstruction                                  |
|  Image-domain transforms                                 |
|  Optical broadcast / routing                             |
+----------------------------+-----------------------------+
                             |
                             | optical image representation
                             v
+----------------------------------------------------------+
|                OPTICAL INTERCONNECT                      |
|                                                          |
|  Waveguide / fiber / photonic interposer                 |
+----------------------------+-----------------------------+
                             |
                             v
+----------------------------------------------------------+
|                OPTICAL OUTPUT MODULE                     |
|                                                          |
|  Spatial conversion / coupling / beam formation          |
+----------------------------+-----------------------------+
                             |
                             v
+----------------------------------------------------------+
|                  OPTICAL DISPLAY                         |
|                                                          |
|  Laser source -> phosphor / projection medium -> light   |
+----------------------------+-----------------------------+
                             |
                             v
                    HUMAN VISUAL SYSTEM
```

## 4. Electronic Domain

The electronic processor remains the primary general-purpose computing system.

It may contain:

- CPU cores;
- conventional GPU compute units;
- tensor/NPU accelerators;
- memory controllers;
- cache hierarchy;
- system memory;
- storage;
- I/O;
- operating-system interfaces.

The important architectural change is that the electronic processor does not necessarily have to construct and transmit a conventional display frame as its final product.

Instead, it provides the photonic subsystem with the information necessary to construct the visual output.

For example:

```
Electronic domain:

Game / application
       -> Scene state
       -> Geometry / physics / lighting information
       -> Neural rendering inputs
       -> Photonic rendering chiplet
```

The exact partition is implementation-dependent.

The architecture does not require all rendering to become photonic.

## 5. Input Data and the E/O Boundary

A critical question that any knowledgeable reader will ask: **does the optical interface on the input side actually win over an electrical one?**

### Input data volumes

For 4K (3840 × 2160) at 120 Hz, the input data for a neural renderer is substantial:

| Data type | Per-pixel | 4K@120 Hz |
|-----------|-----------|-----------|
| G-buffer (position, normal, albedo, roughness, etc.) | 20–40 B | 20–40 GB/s |
| Motion vectors | 4–8 B | 4–8 GB/s |
| Neural features / latent representations | 50–500 MB/frame | 6–60 GB/s |
| **Total input** | | **30–108 GB/s** |

For comparison, raw 4K120 10-bit video — the output of a conventional display chain — occupies approximately 30 Gbit/s (~3.7 GB/s).

**The input traffic is roughly 8–30× larger than the display-link traffic it would replace.**

This is not "the same order of magnitude." The photonic chiplet's input interface carries substantially more data than a conventional display cable.

### Why optical may still be justified

The trade-off depends on the physical context:

- The photonic chiplet is on-package or near-package, so the channel is short (mm to cm), not meters.
- On-package optical interconnect can potentially achieve lower energy per bit than high-speed electrical serial links at equivalent bandwidth.
- The input data does not need to be serialized into a display protocol (HDMI/DP), avoiding protocol overhead, scrambling, and DSC compression.

However, this advantage must be quantified. **The energy-per-bit comparison between on-package optical and on-package electrical interconnect at 30–100 GB/s is an open question and should be addressed explicitly.** [1]

### What does not disappear

E/O conversion does not disappear — it moves from the display cable to the chiplet interface. The architecture changes where conversion happens, not whether it happens.

## 5.1 Shared Memory Hub — Intermediate Buffer Between Compute and Photonic Render
The current GPU memory hierarchy is designed around a linear pipeline: compute writes a framebuffer, display controller reads it. In a hybrid electronic–photonic architecture, this model breaks down — multiple consumers need access to the same scene data simultaneously, at different stages of frame construction, and with different access patterns.

The Problem
During a single frame, several subsystems may require the same data at the same time:

Photonic renderer reads the G-buffer (normals, depth, albedo, motion vectors) to produce the optical image representation.
Neural post-processing (denoising, super-resolution, frame synthesis) reads the same G-buffer plus temporal history.
Adaptive sampling reads variance maps derived from the G-buffer to decide where to cast additional rays.
Compositor reads partially reconstructed image data for UI overlay and final blend.
In a conventional GPU, each of these stages accesses global memory independently — often re-reading the same data from VRAM, incurring full memory bandwidth cost each time. At 4K with path tracing, a single G-buffer pass can consume 30–60 GB/s. If three consumers each read the same G-buffer independently, the effective bandwidth demand triples.

Proposed Structure
We propose an intermediate shared memory hub positioned between the electronic compute domain and the photonic render domain:

┌─────────────────────────────────────────────────┐
│                  Compute (GPU/NPU)              │
│  Ray tracing │ Rasterization │ Neural inference │
└──────────────────────┬──────────────────────────┘
                       │ Write (single producer)
                       ▼
              ┌────────────────────┐
              │  Shared Memory Hub │
              │                    │
              │  G-buffer │ Motion │
              │  Vectors  │ History│
              │  Variance │Features│
              │                    │
              │  Multi-read,       │
              │  single-write      │
              └──┬─────┬─────┬─────┘
                 │     │     │
          ┌──────┘     │     └──────┐
          ▼            ▼            ▼
     Photonic      Neural        Adaptive
     Renderer     Post-Proc      Sampling

Design Principles
Single-write, multi-read. The compute domain writes scene data once. All consumers read from the same physical storage, not from independent copies. This eliminates redundant VRAM traffic.

Coherent snapshot. The hub holds a coherent snapshot of the current frame's intermediate data. Consumers read the same version, avoiding race conditions between, for example, the photonic renderer and the denoiser.

Close to compute. The hub sits on the same package as the compute chiplets — not in external HBM. This reduces read latency from ~100 ns (HBM) to ~10 ns (on-package SRAM or embedded DRAM).

Protocol-agnostic. The hub does not interpret the data — it stores opaque blocks tagged by type (G-buffer, motion, variance) and serves them to any consumer that requests a block by type and tile coordinates.

What This Changes in GPU Architecture
In a conventional GPU, the memory controller is the single arbiter between compute and display. Adding a shared memory hub introduces a second arbitration layer — but one that is simpler, closer to compute, and optimized for broadcast reads rather than sequential pipeline stages.

This is not a fundamental redesign of the GPU. It is a repositioning of the memory hierarchy: instead of one large pool of VRAM serving everything, there is a small, fast, shared buffer that sits between compute and the photonic domain — reducing the bandwidth pressure on both HBM (less redundant traffic) and the E/O interface (data is read once from the hub, not re-fetched from VRAM per consumer).

Bandwidth Impact
If three consumers each require the G-buffer at 30 GB/s, the conventional approach demands 90 GB/s from VRAM. With the shared hub, the VRAM-to-hub transfer is 30 GB/s (one write), and the hub-to-consumer transfers are served from on-package memory at lower energy per bit. The hub does not eliminate the bandwidth — it consolidates it, turning three independent streams into one write plus three local reads.

## 6. Photonic Rendering Chiplet

The photonic chiplet is the central accelerator of the architecture.

It is not intended to replace the entire GPU.

Its purpose is to execute selected image-generation operations for which photonic computation is potentially advantageous.

Candidate workloads include:

- neural rendering;
- neural reconstruction;
- super-resolution;
- ray reconstruction;
- learned denoising;
- neural frame synthesis;
- feature transformations;
- linear image transforms;
- optical filtering;
- image-domain broadcast;
- selected matrix operations.

A simplified representation is:

```
Electronic scene representation
             |
             v
      photonic input
             |
             v
    optical neural network
             |
             v
      optical features
             |
             v
     image reconstruction
             |
             v
      optical image field
```

The photonic chiplet may be integrated into the same package as electronic compute chiplets.

## 7. Chiplet Architecture

A heterogeneous chiplet package is a natural implementation model.

```
+-----------------------------------------------+
|                 PACKAGE                        |
|                                                |
|  +-------+  +-------+  +---------------+       |
|  | CPU   |  | GPU   |  | Memory / I/O  |       |
|  +-------+  +-------+  +---------------+       |
|                                                |
|              +----------------+                 |
|              | Photonic       |                 |
|              | Rendering      |                 |
|              | Chiplet        |                 |
|              +----------------+                 |
|                                                |
|       Electronic / optical interconnect        |
+-----------------------------------------------+
```

This has several advantages.

Different manufacturing processes can be used for different functions. For example:

- advanced CMOS for general compute;
- specialized SRAM/cache technology;
- silicon photonics for optical processing;
- separate laser technology;
- photonic interposer for high-bandwidth communication.

The architecture therefore avoids the requirement that a single semiconductor process must simultaneously be optimal for every function.

Recent research has explored photonic interconnects for chiplet-based GPU architectures, demonstrating the feasibility of this heterogeneous packaging direction. SEECHIP [2] proposes a chiplet-based GPU using photonic links, showing reductions in both execution time and energy consumption versus metallic interconnects across 14 HPC benchmarks.

## 8. Why Photonic Compute Is Used Selectively

A common misunderstanding of this proposal is that it requires a fully photonic processor.

It does not.

The intended architecture is:

```
                 ELECTRONIC
             +-------------------+
             | CPU               |
             | GPU               |
             | memory            |
             | control           |
             | simulation        |
             +--------+----------+
                      |
                      v
                 PHOTONIC
             +-------------------+
             | Neural render     |
             | reconstruction    |
             | transformations   |
             | optical routing   |
             +--------+----------+
                      |
                      v
                    LIGHT
```

The boundary should be determined experimentally.

The guiding criterion is not "electronic versus photonic."

The criterion is:

> Which representation and processing medium minimizes total system cost for a given operation?

## 9. Optical Data Distribution

One potentially important advantage of photonics is optical broadcast.

A conventional electronic architecture may require multiple consumers to read the same data repeatedly:

```
Memory
  |--> Compute block A
  |--> Compute block B
  |--> Compute block C
  +--> Compute block D
```

An optical architecture can potentially distribute one signal through optical splitting or wavelength/spatial multiplexing:

```
                   +--> Block A
                   |
Optical signal ----+--> Block B
                   |
                   +--> Block C
                   |
                   +--> Block D
```

This does not mean that optical splitting is free.

Optical power, insertion loss, noise, detector sensitivity and signal integrity remain engineering constraints.

The architectural opportunity is that broadcast can occur in the physical domain without requiring equivalent duplication of digital memory traffic.

This is an important area for quantitative validation.

## 10. Optical Image Representation

The most important conceptual transition occurs at the output of the photonic renderer.

The system does not necessarily need to produce:

```
3840 x 2160 x RGB x N bits
```

as a conventional digital framebuffer.

Instead, it can produce an optical field:

```
I(x, y, lambda, t)
```

where:

- "x" and "y" describe spatial position;
- "lambda" describes wavelength/spectral content;
- "t" describes time.

This does not imply infinite resolution.

The resulting image remains limited by:

- diffraction;
- optical transfer function;
- numerical aperture;
- aberrations;
- optical noise;
- laser bandwidth;
- modulation depth;
- spatial addressing;
- detector/emitter characteristics;
- phosphor response;
- human visual resolution.

The important difference is that resolution becomes a property of an optical system rather than exclusively a property of a fixed rectangular pixel matrix.

## 11. Continuous Optical Reconstruction

A conventional display samples an image onto discrete pixels.

The optical architecture instead aims to reconstruct the image using a continuous or quasi-continuous optical point-spread function.

A simplified model is:

**Digital pixel representation:**

```
########
########
########
```

**Optical reconstruction (Gaussian PSF):**

```
      .+.
    .+### +.
   +#######+
    .+### +.
      .+.
```

A Gaussian-like point-spread function can act as a physical reconstruction filter.

This creates an important distinction.

**Display sampling:** The optical system can potentially reduce artifacts caused by a hard rectangular pixel aperture.

**Rendering sampling:** It does not automatically eliminate insufficient sampling of the underlying 3D scene.

Therefore the architecture does not claim:

> Optics makes anti-aliasing unnecessary.

The more precise statement is:

> The optical display can perform part of the reconstruction operation physically, potentially reducing the amount of computation required for display-sampling correction.

Neural rendering can then address the remaining scene-sampling problem.

This distinction is fundamental to the architecture.

## 12. Anti-Aliasing as a System Property

Anti-aliasing should be considered as a combination of:

1. scene sampling;
2. reconstruction;
3. display sampling;
4. human visual integration.

The proposed architecture moves part of this chain into optics.

Conceptually:

```
Scene
  -> Neural / electronic sampling
  -> Photonic reconstruction
  -> Optical PSF
  -> Human visual system
```

rather than:

```
Scene
  -> Digital rendering
  -> Digital AA
  -> Digital framebuffer
  -> Pixel sampling
  -> Panel optics
  -> Eye
```

The potential benefit is not that "AA becomes free."

The potential benefit is that some work traditionally performed numerically can become a property of the physical image-forming system.

This should be validated experimentally using controlled spatial-frequency test patterns.

## 13. No Traditional Raster Scan

Traditional flat-panel displays typically depend on a spatially addressed pixel matrix and timing controller.

The proposed optical display does not need to preserve this architecture.

If the optical system forms the image field directly, there is no inherent requirement to scan rows of a TFT matrix.

This can eliminate a class of raster synchronization artifacts.

In particular, conventional scanout tearing can be avoided when the image is formed as a temporally coherent optical field rather than as independently scanned display rows.

However, this does not mean that every temporal artifact becomes impossible.

The system can still have:

- frame-transition discontinuities;
- temporal modulation artifacts;
- laser response limitations;
- phosphor persistence;
- synchronization errors between rendering and optical emission.

The claim is therefore specifically:

> Raster-scan tearing is not an inherent requirement of the proposed display architecture.

## 14. Optical Interconnect

The optical image representation can leave the photonic chiplet through:

- integrated waveguides;
- photonic interposers;
- optical fibers;
- free-space coupling.

The key architectural goal is to avoid converting the image back into a high-speed digital electrical stream merely for transport.

The system becomes:

```
Photonic chiplet
      |
      v
optical waveguide
      |
      v
optical connector / fiber
      |
      v
optical output module
```

This is especially attractive for systems where the compute package and display are physically separated.

## 15. Optical Output Module

A standardized optical output module can serve as the interface between the photonic compute system and different physical display implementations.

Conceptually:

```
Photonic processor
       |
       v
Standard optical interface
       |
       +--> laser-phosphor display
       +--> direct laser projection
       +--> AR optical engine
       +--> VR optical engine
       +--> other optical systems
```

The interface should define:

1. physical optical connection;
2. wavelength conventions;
3. optical power range;
4. spatial representation;
5. temporal modulation;
6. synchronization;
7. calibration;
8. error handling;
9. safety limits.

The objective is not to create another closed proprietary video protocol.

The objective is to define a physical optical representation that can be implemented by multiple vendors.

## 16. Display Architecture

A representative implementation is:

```
Optical input
     -> Optical coupling
     -> Laser source / emitter array
     -> Spatial optical formation
     -> Phosphor / projection medium
     -> Visible image
```

The display no longer requires a conventional TFT pixel matrix for image formation.

Potentially removable components include:

- conventional scaler;
- display-side framebuffer;
- T-CON;
- row/column pixel drivers;
- conventional pixel DAC infrastructure;
- conventional backlight;
- polarizers;
- RGB color-filter matrix.

**However, removing a component does not mean removing its function.** Each listed component performs a real function — timing, addressing, driving, color separation. In the proposed architecture, these functions migrate into the optical domain or are restructured. For example, T-CON timing is replaced by laser scanning control; pixel addressing is replaced by beam steering or emitter array control. The function migrates; it does not disappear.

The actual component reduction depends on the chosen optical implementation.

The architecture therefore should not assume that every proposed display implementation collapses to exactly three physical components.

## 17. Contrast and Dynamic Range

A laser-based optical system can potentially achieve a very low black level because optical emission can be reduced toward zero rather than being generated by a continuously illuminated backlight.

This creates a potential path to very high contrast.

However, practical contrast depends on:

- stray light;
- phosphor persistence;
- optical scattering;
- laser extinction ratio;
- ambient light;
- optical leakage;
- measurement methodology.

Therefore the architecture should treat very high contrast as a potential capability, not as a guaranteed numerical specification.

OLED displays also achieve true black by turning off individual pixels. The potential advantage of the optical architecture is not uniqueness in achieving black, but in combining high contrast with other properties (brightness, color gamut) that are difficult to achieve simultaneously in OLED.

## 18. Color

A photonic display can use wavelength-selective optical sources.

Potential implementations include:

- RGB lasers;
- multiple wavelength channels;
- wavelength-division multiplexing;
- broadband excitation plus spectral filtering;
- multi-primary systems.

The potential advantages include high spectral purity and the possibility of gamuts beyond conventional sRGB.

However, a larger color gamut is not automatically equivalent to better image quality.

The system must consider:

- colorimetric calibration;
- white point;
- spectral power distribution;
- human cone response;
- HDR color volume;
- laser speckle;
- metamerism.

The architecture therefore treats wide gamut as an available design space rather than a fixed performance claim.

## 19. Color and Brightness: Technology-Specific Characteristics

It is important to separate the characteristics of different display technologies that may appear in different parts of the architecture.

| Characteristic | Source technology | Mechanism | Value |
|----------------|-------------------|-----------|-------|
| Color gamut 214% sRGB | TriLite Trixel 3 [3] | Direct RGB laser (narrowband) | TriLite specification |
| Brightness 500 cd/m² (max) | Prysm LPD 6K [4] | 405 nm laser + RGB phosphor (broadband) | Prysm spec sheet |
| Sequential contrast 1,000,000:1 | Prysm LPD 6K [4] | Laser off = true black | Prysm spec sheet |
| Field rate 360 Hz | Prysm LPD 6K [4] | Laser scanning | Prysm spec sheet |
| Lifetime >60,000 hrs | Prysm LPD 6K [4] | Laser-phosphor (photonic excitation) | Prysm spec sheet, to 50% brightness |

**These are characteristics of different technologies. Their advantages do not automatically combine.** The narrowband gamut advantage of direct RGB lasers (TriLite) does not apply to a phosphor-based system (Prysm), which emits broadband light. A system using laser-phosphor inherits Prysm's characteristics; a system using direct RGB lasers inherits TriLite's. These should not be merged into a single column.

The phosphor lifetime claim (>60,000 hrs) is based on photonic excitation, which differs from the electron-beam excitation that caused phosphor degradation in CRTs. However, independent long-term verification of laser-phosphor lifetime under continuous high-power operation is limited and should not be assumed from CRT analogies in either direction.

## 20. Brightness and Power

The potential power advantage comes from architectural simplification, not from the assumption that lasers are inherently efficient.

The total system power must be calculated as:

```
Wall power
   -> electronic compute
   -> photonic compute
   -> laser electrical-to-optical efficiency
   -> optical coupling losses
   -> waveguide/fiber losses
   -> optical conversion efficiency
   -> visible optical power
   -> perceived image
```

This is the correct basis for comparison.

Potential power savings may arise from:

- removal of display-side processing;
- elimination of large pixel-driver networks;
- elimination of conventional backlight architectures;
- reduced digital image movement;
- photonic computation efficiency;
- optical broadcast;
- direct optical image formation.

A target such as "75% lower display power" should therefore be treated as a system-level hypothesis to be measured, not a guaranteed result.

## 21. Latency

The architecture potentially removes several buffering and serialization stages.

A conceptual latency chain is:

```
Scene update
   -> electronic scheduling
   -> photonic neural inference
   -> optical propagation
   -> optical emission
   -> visual response
```

rather than:

```
Scene update
   -> render
   -> framebuffer
   -> post-processing
   -> scanout
   -> serialization
   -> transport
   -> monitor buffering
   -> scaler
   -> T-CON
   -> pixel scan
   -> light
```

The potential latency reduction is therefore real as an architectural hypothesis.

The correct measurement boundary is:

> Time from a defined scene-state change to the corresponding measured optical change at the display output.

This is **display-chain latency**, not end-to-end latency. End-to-end latency includes game logic, scene simulation, and GPU render time, which dominate the total and are not addressed by this architecture. A 120 Hz render cycle alone takes ~8 ms; the display chain adds several milliseconds on top.

Modern gaming monitors at 120–240 Hz achieve display-chain latency (from frame ready at GPU output to photons) in the range of a few milliseconds, not 10–100 ms. The baseline for comparison should be these modern values, not legacy figures.

An end-to-end latency below 1 ms is not achievable without racing the beam or similar scan-locked rendering, regardless of display technology. The architecture's claim is specifically about **display-chain latency**: the segment from photonic renderer output to visible photons.

## 22. Bandwidth

A common shorthand is to say that "light has unlimited bandwidth."

That is not technically accurate.

The correct statement is:

> Optical systems are not intrinsically constrained by the fixed bitrate of a display protocol such as HDMI or DisplayPort. Their usable bandwidth is instead determined by the physical optical system.

Relevant limits include:

- modulation bandwidth;
- wavelength count;
- spatial channels;
- temporal channels;
- signal-to-noise ratio;
- detector/emitter bandwidth;
- coupling efficiency;
- optical nonlinearities;
- dynamic range.

This distinction is important because it turns the architecture from a theoretical claim into an engineering problem that can be quantified.

For reference: HDMI 2.1 supports 48 Gbit/s signalling (42.6 Gbit/s max data rate). DisplayPort 2.1 supports up to 80 Gbit/s. These are the conventional baselines.

## 23. Resolution

The proposed architecture does not have a fixed pixel count in the same sense as an LCD panel.

Instead, image resolution becomes a function of the optical system.

Relevant parameters include:

- wavelength;
- numerical aperture;
- optical transfer function;
- spot size;
- spatial modulation capability;
- aberration;
- phosphor grain/response;
- optical geometry;
- viewing distance.

Thus, "floating resolution" should be understood as:

> Continuously configurable optical spatial resolution within the limits of the optical transfer function.

A 4K-equivalent image and an 8K-equivalent image do not necessarily require two physically different pixel matrices. They may instead correspond to different optical spatial-frequency requirements.

## 24. What the Architecture Potentially Removes

The goal is not to claim that every existing component is always unnecessary.

Rather, the architecture identifies components that may become unnecessary when the optical representation is preserved.

**Compute side — potentially reduced:**

- conventional display controller;
- scanout engine;
- high-speed display serializer;
- DSC specifically used for display transport;
- HDMI/DisplayPort PHY;
- protocol-specific display logic.

**Interconnect — potentially reduced:**

- high-speed electrical differential pairs;
- retimers;
- redrivers;
- protocol conversion.

**Display side — potentially reduced:**

- scaler;
- display framebuffer;
- T-CON;
- row/column drivers;
- conventional pixel DAC architecture;
- conventional backlight;
- polarizers;
- color-filter matrix.

**Important:** Each conventional layer was added to solve a real problem — interoperability (HDMI), bandwidth limits (DSC), pixel timing (T-CON), color separation (color filters). The intent is not "the industry does things stupidly" but "here is a specific layer, here is its function, here is a potentially simpler way to perform it." The exact reduction depends on implementation.

## 25. What Does NOT Disappear

The architecture does not eliminate the need for:

- memory;
- control logic;
- calibration;
- thermal management;
- power electronics;
- safety systems;
- synchronization;
- optical alignment;
- signal conditioning;
- manufacturing tolerances;
- error correction where necessary.

It also does not eliminate the need for general-purpose electronic computation.

This is a heterogeneous architecture, not a claim that electronics are obsolete.

## 26. Addressing Budget

This is arguably the central technical challenge of the entire proposal.

### The numbers

For 4K (3840 × 2160 = 8,294,400 pixels) at 240 Hz:

- If RGB is modulated sequentially on a single beam: 8.29M × 240 Hz ≈ **2 Gspots/s**
- If RGB is modulated in parallel on three beams: 8.29M × 240 Hz × 3 ≈ **6 Gspots/s**

The difference depends on whether the three laser wavelengths share one scanning path or use separate modulators.

### Variant A — Beam scanning

A single scanning beam traces the image point by point, like a laser CRT.

**Strengths:**

- Only one modulator chain needed.
- Directly compatible with laser-phosphor display (Prysm uses a similar principle at 360 Hz with 12.9M pixels [4]).
- MIT photonic ski-jump demonstrates chip-scale beam scanning at 68.6 Mspots/s·mm², equivalent to a 1-megapixel display at 100 Hz from 1.5 mm² [5]. For 2 Gspots/s, this rough metric suggests a footprint on the order of 29 mm² — a very rough estimate that does not account for 10-bit modulation or RGB sequencing.

**Weaknesses:**

- Requires very fast beam steering (kHz mechanical resonances, Lissajous patterns).
- Sweep artifacts, brightness uniformity, and phosphor timing remain challenges.
- 10-bit intensity modulation at GHz spot rates is not demonstrated.

### Variant B — Emitter array

A 2D array of independently controllable optical emitters, analogous to a pixel matrix but with optical sources.

**Strengths:**

- Parallel addressing, no scan artifacts.
- Conceptually closer to existing display manufacturing.

**Weaknesses:**

- Requires millions of individually addressable modulators — essentially rebuilding a pixel matrix in the optical domain.
- Current photonic chips operate with tens to hundreds of ports, not millions.
- Driver and routing complexity may reproduce the T-CON and row/column driver architecture the proposal seeks to simplify.

### Variant C — Tiled scanner array

An array of small scanning modules, each responsible for a tile of the display (e.g., 4×4 tiles of 1080p each).

**Strengths:**

- Reduces per-scanner speed requirement by the tile count.
- Each scanner operates at manageable rates.
- Combines parallelism of Variant B with simplicity of Variant A.
- Prysm's architecture uses multiple laser engines scanning different regions of a large display [4], suggesting this is practical.

**Weaknesses:**

- Tile boundaries may produce seams or brightness non-uniformity.
- Synchronization between scanners adds complexity.
- Total component count is higher than Variant A.

### Status

No variant is selected. This is an open architectural question. The choice depends on engineering trade-offs that require experimental validation. The MIT ski-jump result [5] and Prysm's multi-engine scanning [4] both provide evidence for Variant A/C, while the scaling challenges of Variant B are well-documented in photonic computing literature.

## 27. Transitional Architecture

A major advantage of the concept is that full photonic rendering is not required for the first prototype.

A transitional system can use an existing electronic GPU:

```
Existing GPU
     -> Electronic-to-optical conversion
     -> Optical transport
     -> Optical output module
     -> Laser/phosphor or projection display
```

This isolates the display architecture from the photonic compute problem.

The first experiment therefore does not need to prove:

> Photonic neural rendering is commercially viable.

It only needs to test:

> Can an optical image representation be transported and converted into a high-quality display image without reconstructing a conventional digital display pipeline?

If successful, the photonic rendering chiplet can be introduced later.

**Note:** In a transitional architecture using DMD or LCoS as the spatial modulator, the modulator has its own controller (analogous to a T-CON), and contrast is limited by the modulator's extinction ratio, not by laser on/off alone. Claims about "no T-CON" and "infinite contrast" do not apply to the transitional device — they are targets for a fully optical architecture.

## 28. Experimental Roadmap

The architecture should be validated in stages.

### Stage 1 — Optical transport

Demonstrate:

```
Electronic image source
        -> E/O conversion
        -> Optical fiber
        -> Optical display
```

Measure:

- latency;
- bandwidth;
- SNR;
- optical dynamic range;
- color fidelity;
- spatial resolution.

### Stage 2 — Optical reconstruction

Compare:

- Pixel reconstruction vs. optical PSF reconstruction

Use:

- checkerboards;
- diagonal lines;
- text;
- subpixel patterns;
- moving edges;
- high-frequency textures;
- temporal patterns.

Measure the resulting modulation transfer function.

### Stage 3 — Spatial optical addressing

Demonstrate:

```
I(x, y, lambda, t)
```

generation from an optical input.

This is one of the most important technical milestones. See Section 26 (Addressing Budget) for the open variants.

### Stage 4 — Neural rendering chiplet

Introduce a photonic neural network for a controlled workload.

Measure:

- energy per operation;
- latency;
- optical loss;
- precision;
- error rate;
- thermal stability;
- scaling behaviour.

Recent experimental work has demonstrated programmable photonic neural networks capable of processing image data directly. ACCEL [1] achieves 72 ns per frame for classification tasks (Fashion-MNIST, 3-class ImageNet), and OPCA [6] achieves 6 ns response time for image processing. **However, these are classification and sensing tasks on small images, not 4K frame rendering.** They serve as proof of concept for photonic computation, not as ready components for the proposed pipeline.

### Stage 5 — Direct photonic rendering-to-display path

Finally:

```
Electronic control
       -> Photonic neural rendering
       -> Optical image field
       -> Optical interconnect
       -> Optical display
```

No intermediate digital framebuffer is required between the photonic renderer and optical output.

## 29. Critical Engineering Questions

The concept has several important open problems.

### 29.1 Spatial addressing

How many independently controllable optical degrees of freedom can be produced? See Section 26.

### 29.2 Optical power budget

How much optical power is required after splitting, coupling, propagation, modulation, and conversion?

### 29.3 Precision

Photonic computation is often analog. Effective precision is typically 4–8 bits with noise and thermal drift.

This is potentially **insufficient for smooth HDR gradients** (risk of banding at 8-bit precision in high-dynamic-range content). This is a fundamental limitation, not an engineering nuisance.

The system must characterize:

- noise;
- quantization;
- phase error;
- amplitude error;
- thermal drift;
- manufacturing variation.

### 29.4 Nonlinear operations

Linear optical transformations are relatively natural. Neural networks also require nonlinearities.

The architecture therefore needs an efficient photonic/electronic strategy for nonlinear activation functions and control.

### 29.5 Memory

Photonics does not automatically solve memory. Large persistent state remains primarily an electronic problem unless a suitable photonic memory technology is used.

The proposed architecture therefore deliberately leaves memory in the electronic domain wherever appropriate.

### 29.6 Calibration

An optical system may require calibration across wavelength, temperature, emitter intensity, phase, spatial alignment, and phosphor response. Calibration infrastructure must be considered part of the architecture.

## 30. Where the Real Architectural Novelty Lies

The novelty of this proposal should not be interpreted as:

> Photonic neural networks are new.

They are not.

> Optical displays are new.

They are not.

> Chiplets are new.

They are not.

The proposed architectural contribution is the composition:

```
Electronic general-purpose compute
  + Photonic neural rendering
  + Optical image transformation
  + Optical interconnect
  + Direct optical display formation
```

with the deliberate objective of avoiding unnecessary electronic/digital representations between photonic rendering and visual output.

The architecture therefore changes the location of the electronic/optical boundary.

That boundary is the primary subject of this proposal.

## 31. Where I Am Probably Wrong

The following are the areas where this proposal is most likely to fail or require significant revision. Each is numbered for discussion in GitHub Issues.

### 31.1 Addressing (Section 26)

The 2–6 Gspots/s requirement for 4K@240 Hz may be beyond what any current scanning or emitter technology can achieve with 10-bit precision. The proposal may need to accept lower resolution, lower bit depth, or a tiled architecture (Variant C) that has not been demonstrated at scale.

### 31.2 Precision and HDR (Section 29.3)

Analog photonic computation at 4–8 effective bits may be fundamentally incompatible with smooth HDR gradients. If 10-bit output is required, the system may need hybrid analog/digital encoding that negates some of the architectural simplification.

### 31.3 What photonics adds for latency and tearing (this section)

Latency and tearing are already addressed by existing digital techniques: scanout without buffering, racing the beam, bufferless output. **What does photonics specifically add beyond these techniques?**

The honest answer may be: **not much, at least in the near term.** The latency advantage comes primarily from the scanning architecture, not from photonics per se. A laser-phosphor display driven by a conventional digital signal could achieve similar display-chain latency. The photonic rendering chiplet's contribution to latency is unproven.

If this is the case, the proposal should acknowledge that the latency benefit is a property of the display architecture (laser scanning, no framebuffer), not of photonic computation. This would narrow the scope but make the claim more defensible.

### 31.4 Input traffic vs. output traffic (Section 5)

Input traffic to the photonic chiplet (30–108 GB/s) is 8–30× larger than the display-link traffic it replaces. **Why does an optical interface on the input side win over an electrical one?**

This is unproven. The advantage may be in energy-per-bit at short distances, but this requires quantitative comparison with on-package electrical interconnect (e.g., UCIE, AIB). If electrical interconnect is cheaper and equally fast at these distances, the optical input interface may not be justified.

### 31.5 T-CON and driver functions (Section 16)

Removing the T-CON does not remove the need for timing control. The function migrates to laser scanning control or modulator array control. If the replacement is equally complex, the architectural simplification is illusory.

### 31.6 Phosphor lifetime (Section 19)

The 60,000-hour claim is from the manufacturer (Prysm [4]) and is based on a different excitation mechanism than CRT phosphor degradation. Independent long-term verification under continuous high-power laser excitation is limited.

### 31.7 Simulation as evidence (simulation/)

The simulation results (93.5% latency reduction, 76.7% power reduction) are derived from the author's own assumptions about component behavior. They are an **architectural illustration**, not experimental proof. They cannot be used as evidence for the proposal's feasibility.

### 31.8 Each conventional layer solved a real problem

HDMI solves interoperability. DSC solves bandwidth limits. T-CON solves pixel timing. Color filters solve color separation. The proposal must show not just that these can be removed, but that their functions are adequately replaced. If any replacement is inferior, the layer should not be removed.

## 32. Architectural Principle

The entire proposal can be reduced to the following rule:

> Use electronics to decide what image should exist. Use photonics to perform the transformations that benefit from optical parallelism. Once the information has become an optical image representation, keep it optical until it becomes visible light.

This is the core idea.

## 33. Potential System-Level Benefits

The following are potential consequences to be experimentally evaluated.

**Lower data movement** — less conversion between compute, memory and display representations.

**Lower display-interface complexity** — no requirement for a conventional serialized digital video protocol between renderer and display.

**Reduced buffering** — a direct optical path can potentially eliminate several frame-storage stages.

**Optical broadcast** — one optical representation can potentially be distributed to multiple processing paths without equivalent digital copying.

**Physical image reconstruction** — some reconstruction behaviour can be implemented by the optical PSF rather than numerical filtering.

**Modular hardware** — photonic processing can exist as a chiplet rather than requiring an entirely photonic processor.

**Display simplification** — the display can become primarily an optical engine rather than a computer attached to a panel.

**Scalable optical interconnect** — optical transport can potentially provide high bandwidth without the same electrical routing constraints.

## 34. Potential Performance Envelope

The following values should be treated as architectural targets / hypotheses, not demonstrated specifications.

| Parameter | Potential direction |
|-----------|-------------------|
| Display-chain latency | Potentially sub-millisecond (display chain only, not end-to-end) |
| Optical transport bandwidth | Potentially multi-Tbit/s class depending on implementation |
| Photonic compute efficiency | Potentially below conventional electronic MAC energy for suitable workloads |
| Display power | Potential reduction depending on laser and conversion efficiency |
| Contrast | Potentially very high due to optical extinction (not unique vs. OLED) |
| Color gamut | Potentially beyond sRGB (technology-dependent: narrowband laser vs. phosphor) |
| Spatial resolution | Determined by optical transfer function rather than fixed pixel count |
| Display-side compute | Potentially greatly reduced |
| Raster synchronization | Potentially eliminated as a structural requirement |
| Digital video interface | Potentially eliminated from the rendering-to-display path |

These numbers are deliberately presented as areas for investigation, not as claims of achieved performance.

## 35. Comparison of Architectural Philosophies

**Conventional evolution:**

```
Existing architecture
  -> optimize component
  -> add feature
  -> add controller
  -> add protocol
  -> add buffer
  -> add algorithm
  -> repeat
```

**Proposed approach:**

```
Existing architecture
  -> identify unnecessary representation
  -> remove conversion
  -> move suitable operation into physical domain
  -> preserve representation
  -> simplify downstream system
```

The difference is therefore architectural rather than merely technological.

## 36. Why This May Be Timely

The concept depends on technologies that historically existed in separate research communities:

- photonic neural computing;
- silicon photonics;
- optical interconnects;
- chiplet packaging;
- laser display systems;
- optical signal processing;
- neural rendering.

These technologies are increasingly converging.

Recent work has demonstrated increasingly capable photonic neural processors, including architectures designed to process 2D image data directly. ACCEL [1] and OPCA [6] show nanosecond-scale photonic computation for vision tasks. At the interconnect level, SEECHIP [2] demonstrates photonic links for chiplet-based GPUs. MIT's photonic ski-jump [5] demonstrates chip-scale beam scanning for display and projection.

This does not prove the complete architecture.

It does, however, mean that the proposal can be evaluated as a systems-integration problem rather than requiring every underlying technology to be invented from scratch.

## 37. The Key Research Question

The central research question is not:

> Can a computer be made entirely photonic?

It is:

> How much of the graphics pipeline can remain in the optical domain once neural rendering has produced the image representation?

A second question follows:

> What is the optimal boundary between electronic and photonic computation for a practical graphics processor?

These questions are experimentally testable.

## 38. Falsifiable Predictions

A useful architecture should make testable predictions.

The proposal predicts that, for selected workloads:

1. Optical broadcast can reduce data movement relative to equivalent electronic replication.
2. Photonic neural computation can reduce latency or energy for suitable linear-algebra-heavy workloads.
3. Optical image reconstruction can replace part of numerical display reconstruction.
4. Direct optical transport can remove several conventional display-interface stages.
5. A display without a conventional rasterized pixel matrix can produce useful high-resolution images.
6. A heterogeneous electronic/photonic chiplet can provide a more practical path than a fully photonic processor.
7. End-to-end system efficiency must be evaluated at wall-plug level rather than by quoting photonic MAC efficiency alone.

These predictions can be independently tested.

## 39. What Would Constitute Success?

The concept does not require every theoretical advantage to materialize.

A successful implementation would demonstrate that:

```
Electronic compute
  -> Photonic rendering
  -> Optical image
  -> Optical transport
  -> Optical display
```

can outperform or simplify an equivalent conventional path for at least one meaningful workload.

Success could therefore be demonstrated by any combination of:

- lower latency;
- lower energy;
- reduced data movement;
- reduced hardware complexity;
- improved optical reconstruction;
- higher bandwidth;
- improved display quality;
- simpler manufacturing.

The architecture should be judged by measurable system-level results rather than by individual component specifications.

## 40. Broader Implication

The proposal is an example of a more general engineering principle.

When a system becomes increasingly complicated, the natural response is often to optimize each component.

But sometimes the greater opportunity is to question whether the intermediate representation itself is necessary.

In graphics, the final output is light. Therefore a potentially unnecessary chain is:

```
light -> digital representation -> processed digital representation
  -> serialized digital representation -> reconstructed digital representation -> light
```

The proposed architecture asks whether a significant portion of that loop can be removed.

## 41. Historical Context: CRT and Flat Panels

The CRT was, in many ways, the simple analog chain this proposal envisions: scene -> electron beam -> phosphor -> light. No framebuffer, no digital serialization, no T-CON.

Flat panels (LCD, OLED) replaced CRTs not because they were simpler — they are architecturally more complex — but because they solved physical problems: size, weight, geometry, power, manufacturing cost.

The lesson is relevant: a simpler architecture does not automatically win. The proposed optical architecture must demonstrate not just simplicity but practical advantages in measurable dimensions: latency, power, brightness, contrast, or manufacturing cost.

## 42. Final Architecture

The target architecture can therefore be summarized as:

```
                    ELECTRONIC DOMAIN
 +---------------------------------------------------+
 |                                                   |
 |  CPU / GPU / NPU / Memory / Simulation / Control  |
 |                                                   |
 +-----------------------+---------------------------+
                         |
                         | neural rendering inputs
                         | and control
                         v
                  PHOTONIC CHIPLET
 +---------------------------------------------------+
 |                                                   |
 |  Neural rendering                                |
 |  Reconstruction                                  |
 |  Image transformation                            |
 |  Optical broadcast                               |
 |                                                   |
 +-----------------------+---------------------------+
                         |
                         | optical image representation
                         v
                 OPTICAL INTERCONNECT
 +---------------------------------------------------+
 |  Photonic interposer / waveguide / optical fiber  |
 +-----------------------+---------------------------+
                         |
                         v
                  OPTICAL OUTPUT
 +---------------------------------------------------+
 |  Spatial optical conversion / emitter system      |
 +-----------------------+---------------------------+
                         |
                         v
                  OPTICAL DISPLAY
 +---------------------------------------------------+
 |  Laser / phosphor / projection / optical field    |
 +-----------------------+---------------------------+
                         |
                         v
                        EYE
```

## 43. Conclusion

This proposal does not attempt to replace electronics with photonics.

It attempts to use each technology where its physical properties are most appropriate.

Electronics remain responsible for general computation, control, memory and system logic.

Photonics is introduced where massive parallelism, optical propagation, signal broadcast and neural linear algebra can provide an advantage.

Most importantly, the resulting image does not necessarily need to be converted back into a conventional digital framebuffer before reaching the display.

The architecture therefore seeks to remove a class of intermediate representations rather than optimize them indefinitely.

The essential sequence is:

```
COMPUTE ELECTRONICALLY
  -> RENDER PHOTONICALLY
  -> TRANSFORM OPTICALLY
  -> TRANSMIT OPTICALLY
  -> DISPLAY OPTICALLY
```

The central proposition is simple:

> If the final information is light, there is value in asking how early in the pipeline it can become light — and how late it can remain light.

The technologies required for individual stages already exist in various forms.

The research challenge is to determine whether their deliberate integration can produce a superior system architecture.

That is the purpose of this proposal.

## Status and Scope

This document is an architectural proposal.

Performance numbers are targets, estimates or potential envelopes unless explicitly identified as experimentally demonstrated.

The simulation in `simulation/latency_model.py` is an **architectural illustration**, not experimental proof. The numbers are derived from the author's assumptions about component behavior, not from measured photonic hardware.

The purpose of the project is to establish a coherent architecture and identify experimentally testable engineering paths toward implementation.

It is not a claim that a complete commercial system already exists.

## 44. Existing Technologies Supporting the Architecture

| Technology | Institution | Year | Status | Key metric | Relevance |
|-----------|-----------|------|--------|-----------|-----------|
| ACCEL [1] | Tsinghua University | 2023 | Published, Nature | 72 ns/frame, 3000× faster than GPU for classification | Photonic computation for vision |
| OPCA [6] | Tsinghua University | 2024 | Published, Optica | 6 ns response, 100B pixel bandwidth | End-to-end optical image processing |
| SEECHIP [2] | University of Otago | 2023 | Published, ICPP | Photonic inter-chiplet network for GPU | Chiplet-scale optical interconnect |
| MIT ski-jump [5] | MIT / MITRE | 2025 | Published, Nature | 68.6 Mspots/s·mm², chip-scale beam scanning | Free-space beam scanning (Variant A) |
| TriLite Trixel 3 [3] | TriLite Technologies | 2023–2026 | Prototype, engineering samples | 214% sRGB, <1.5 g, <1 cm³ | Direct RGB laser beam scanning |
| Prysm LPD 6K [4] | Prysm Systems | 2019+ | Commercial product | 360 Hz, 1,000,000:1, 12.9M pixels, >60,000 hrs | Laser-phosphor display |
| Brilliance RGB [7] | Brilliance RGB | 2026 | Startup, €6M funded | Integrated RGB laser chips for AR | Photonic laser source integration |

**Important scale notes:**

- ACCEL's 3000× speedup is for **classification of small images** (Fashion-MNIST, 3-class ImageNet), not 4K frame rendering.
- OPCA's 6 ns response is for **sensing and computation on chip-scale images**, not 4K output.
- MIT ski-jump is **free-space beam scanning**, not chip-to-fiber coupling. Its metric (68.6 Mspots/s·mm²) is directly relevant to addressing Variant A.
- TriLite Trixel 3 is a **prototype** (Best Prototype Award at Display Week I-Zone 2026 [3]), with engineering samples available since March 2024. It is not a shipping commercial product.
- Prysm LPD 6K is a **commercial product** for large-format workplace displays, not a consumer gaming monitor.

## 45. Economics and Distributed Research Path

### 45.1 Cost tiers (order-of-magnitude estimates)

| Stage | Activity | Estimated cost | Notes |
|-------|----------|---------------|-------|
| 1–2 | Optical transport + PSF reconstruction | $10K–50K | University lab, off-the-shelf components |
| 3 | Spatial optical addressing | $100K–500K | Optical engineering lab, custom components |
| 4 | Neural rendering chiplet | $1M–5M | Photonic chip fabrication, packaging |
| 5 | Full path integration | $5M–15M | Multi-domain integration, fab time |

**These are hardware-only, order-of-magnitude estimates excluding staff, facility, and overhead.** Actual costs may differ significantly.

### 45.2 Distributed research model

No single laboratory needs to build the entire system.

Each referenced group already holds one piece:

- Tsinghua (ACCEL, OPCA): photonic computation for vision.
- MIT (ski-jump): chip-scale beam scanning.
- Prysm: laser-phosphor display manufacturing.
- TriLite: compact laser beam scanning for AR.
- Brilliance: integrated RGB laser sources.

This proposal provides an **integration framework** — an architectural specification for how these pieces could compose. Under CC BY 4.0, any group can use, modify, and publish results based on this architecture.

The model is analogous to open-source software: no single entity builds the entire stack, but shared interface specifications allow independent development.

### 45.3 Economic drivers

| Component removed | Cost avoided |
|-------------------|-------------|
| HDMI licensing | $10K/year + $0.05/port |
| DisplayPort PHY | Silicon area, power |
| HDCP | Licensing, complexity |
| DSC | Compression engine, latency |
| T-CON | Display-side controller |
| High-speed cable | Copper, connectors, shielding |

These are illustrative, not exhaustive. Actual savings depend on implementation.

### 45.4 Transitional device as commercial entry point

A transitional device (FPGA + fiber + laser-phosphor display) can deliver display advantages — brightness, contrast, low display-chain latency — before photonic rendering is mature. This is a potential product before the full research platform.

### 45.5 Open licensing strategy

The architecture is published under CC BY 4.0. The strategic intent is:

- **Value is in the integration**, not in patenting individual components.
- **Defensive publication** prevents competitors from patenting the architectural composition.
- **Equal access** — any party can implement, commercialize, or build on the architecture.

For prior art purposes, a Zenodo DOI and/or arXiv preprint is recommended to establish a timestamped, citable record beyond a GitHub repository.

## References

[1] Chen, Y. et al. "All-analog photoelectronic chip for high-speed vision tasks." *Nature* 623, 48–57 (2023). https://doi.org/10.1038/s41586-023-06558-8

[2] Zhang, H. et al. "SEECHIP: A Scalable and Energy-Efficient Chiplet-based GPU Architecture Using Photonic Links." *ICPP 2023*. https://dl.acm.org/doi/10.1145/3605573.3605626

[3] TriLite Technologies. "Wrapping up Display Week 2026 as an Award Winner." https://www.trilite-tech.com/wrapping-display-week-2026-as-an-award-winner-trixel-lbs/ — Best Prototype Award, Display Week I-Zone 2026. Engineering samples available since March 2024.

[4] Prysm Systems. "LPD 6K Series II Specification Sheet." https://www.prysm.com/hubfs/Prysm_LPD_6K_225_Series_II_Spec_Sheet_%20552-0009x-00_Rev_01_2019_10_11.pdf — 6864×1872, 12.9M pixels, 360 Hz, 1,000,000:1 contrast, >60,000 hrs.

[5] Wen, H. et al. "Nanophotonic waveguide chip-to-world beam scanning." *Nature* (2025). https://doi.org/10.1038/s41586-025-10038-6 — 68.6 Mspots/s·mm², piezoelectric cantilever beam scanner, 200 mm CMOS foundry.

[6] Wu, W. et al. "Parallel photonic chip for nanosecond end-to-end image processing, transmission, and reconstruction." *Optica* 11, 873 (2024). https://www.optica.org/about/newsroom/news_releases/2024/june/photonic_chip_integrates_sensing_and_computing_for_ultrafast_machine_vision/

[7] Brilliance RGB. "Brilliance Raises Millions to Advance Its Laserchips for AR." Optica Corporate Member News, March 2026. https://www.optica.org/about/newsroom/corporate_member_news/2026/brilliance_raises_millions_to_advance_its_laserchips_for_ar/ — €6M funding, silicon nitride PIC with integrated RGB laser sources.

## License

Creative Commons Attribution 4.0 International (CC BY 4.0).

The architecture is intentionally published openly so that it can be studied, implemented, improved and commercialized by others.

The objective is to make the architectural concept available rather than restrict its development through proprietary ownership.

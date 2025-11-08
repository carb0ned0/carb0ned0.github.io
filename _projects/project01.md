---
layout: page
title: Flexible Microstrip Antenna
description: A from-scratch design and simulation of a flexible microstrip patch antenna for applications in wearable technology and conformal electronics.
# img: assets/img/12.jpg
importance: 1
category: work
related_publications: false
---

## Overview

As electronic systems become increasingly integrated into our daily lives—through wearable devices, smart textiles, and the Internet of Things (IoT)—the demand for components that are non-rigid, lightweight, and can conform to curved surfaces has surged. Traditional antennas, built on rigid PCB substrates like FR-4, fail in these applications as they are brittle and cannot bend.

The goal of this project was to design, model, and simulate a flexible microstrip patch antenna from scratch. As the sole design engineer, I handled the entire pipeline: from theoretical research and material selection to parametric simulation, optimization, and analysis of the final results. The primary challenge was to achieve reliable performance (stable frequency, good impedance matching) while using a flexible substrate that is inherently sensitive to physical deformation. The design incorporated a hexagonal patch array with slots and a split-ring resonator to enable multiband potential and miniaturization, simulated over 1–9 GHz.

## Research

The design process was an iterative loop of research, calculation, simulation, and optimization.

1. **Literature Review & Specification:** I began by researching common flexible substrates (like Polyimide/Kapton, PDMS, PET, and Rogers RT/duroid) and their electromagnetic properties (dielectric constant ε_r, loss tangent tanδ). I selected PET due to its excellent flexibility, low cost, and suitable dielectric properties (ε_r = 3.5, low conductivity 1e-12 S/m). Copper was chosen as the conductor for its high conductivity (5.96e7 S/m) and ease of deposition. I then defined the key design constraints:
   - **Target Frequency (f_r):** Approximately 1.9 GHz, suitable for LTE or ISM bands.
   - **Substrate:** PET, ε_r = 3.5, thickness h = 0.4 mm.
   - **Conductor:** Pure copper, thickness = 0.035 mm.
   - **Feed Type:** Microstrip line for simple integration.

2. **Theoretical Modeling:** Using standard transmission line models and patch antenna formulas, I calculated initial dimensions for the patch width (W), length (L), and feedline geometry. For a hexagonal array with slots, I incorporated split-ring resonator elements to enhance bandwidth and reduce size. These calculations provided a baseline, but simulations were essential for accuracy given the complex geometry.

3. **Iterative Simulation & Optimization:** I built the 3D model in CST Studio Suite, a powerful electromagnetic solver using the Finite Integration Technique (FIT). The initial simulation (v5 draft) showed resonances around 1.9 GHz, 4.8 GHz, and others, but with shallow S11 dips. This led to an iterative optimization loop:
   - I performed parameter sweeps on variables like slot sizes (0.5–2 mm), hexagon dimensions, and substrate height.
   - Adjustments focused on improving S11 (return loss) as per professor feedback, mapping how geometric changes affected impedance matching.
   - After several versions, the final v6 design with height constraints achieved a deep S11 dip at the target frequency.

## Implementation

The final solution is a fully simulated and optimized antenna design, ready for fabrication.

- **Final Design Artifact:** The core artifact is the 3D model in CST, featuring a 20 mm x 30 mm PET substrate with a copper hexagonal patch array, slots for bandwidth enhancement, and a split-ring resonator for resonance tuning. The microstrip feed is 3 mm wide, with precise gaps (e.g., 2 mm insets) for 50 Ω matching.

- **Key Performance Metrics:** The simulation results highlight the design's matching but reveal efficiency challenges. The S11 plot is the primary metric.
  - **Return Loss (S11):** The final design achieved a peak S11 of -26.5 dB at 1.896 GHz. This value, well below the -10 dB threshold, indicates excellent impedance matching and minimal signal reflection (less than 0.5% power reflected).
  - **Bandwidth:** The -10 dB bandwidth is narrow, approximately 100–200 MHz around the primary resonance, sufficient for niche narrowband applications but sensitive to detuning.
  - **VSWR:** Near 1.1 at resonance, confirming low mismatch.
  - **Efficiency:** Radiation efficiency peaks at -22.97 dB (~0.5%) and total efficiency at -33.07 dB (~0.05%) at 1.896 GHz, indicating significant losses likely due to the compact size and thin materials.
  - **Power Balance:** For 0.5 W input, radiated power is low (~0.0002 W), with minimal dielectric/metal losses but high reflection off-resonance.

A key trade-off in this flexible design is efficiency versus compactness. The electrically small size (relative to wavelength ~158 mm at 1.9 GHz) leads to high ohmic losses. I simulated adaptive meshing, converging after 5 passes with Delta S ≈ 0.036, 170,308 mesh cells, and a solve time of 3,749 seconds, ensuring reliable results.

## Results

This project successfully validated a design methodology for a flexible microstrip patch antenna. The final simulated prototype met the matching goals but highlighted areas for efficiency improvement, proving the feasibility for non-rigid applications like wearables.

- **Key Outcome:** A fabrication-ready CST model for a flexible ~1.9 GHz antenna with multiband potential.
- **Lessons Learned:** The split-ring resonator and slots greatly influenced resonance, but tiny adjustments (e.g., height constraints) impacted efficiency profoundly, emphasizing precision in simulations. The low efficiency underscores the challenges of miniaturization in flexible designs.
- **What I'd Do Differently:** Next steps include fabrication using techniques like inkjet printing or etching, followed by empirical testing with a Vector Network Analyzer (VNA). This would validate simulations against real-world performance, including bending effects which could shift frequency due to permittivity changes.

## Reflection

- **Technology Stack:**
  - **Simulation:** CST Studio Suite.
  - **Calculations & Data Analysis:** Built-in CST tools for post-processing.

- **Why this stack?** CST excels in time-domain simulations for broadband analysis, using FIT to solve Maxwell's equations accurately for complex structures like slotted hexagons.

- **Challenges & Future Work:** The computational intensity of iterations was a hurdle, with each run taking minutes to hours. Future work could integrate this antenna into a wearable system, analyzing body proximity effects (e.g., SAR) and optimizing for higher efficiency through thicker substrates or alternative materials like graphene.

> Code and design files are forthcoming.

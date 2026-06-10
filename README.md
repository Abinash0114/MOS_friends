# Project Proposal: IEEE SSCS Chipathon 2026

## 1. Project Title
**Design and Implementation of a High-Frequency, Low-Power 2nd-Order Gm-C Kerwin-Huelsman-Newcomb (KHN) Biquad Filter Subsystem in 180nm GF180MCU Open-Source CMOS Technology**

---

## 2. Abstract
This proposal presents the design, simulation, and layout implementation of a continuous-time 2nd-order Gm-C based Kerwin-Huelsman-Newcomb (KHN) biquad filter targeted for the open-source GF180MCU 180nm CMOS technology node. Operating from a 3.3V supply rail to maximize dynamic range, the filter architecture leverages highly optimized Operational Transconductance Amplifiers (Gm cells) as its primary building blocks. 

The KHN topology is chosen for its unique capability to simultaneously synthesize Lowpass (LP), Bandpass (BP), and Highpass (HP) transfer functions from a single structure without altering the core hardware configuration. Designed to achieve an individual cell Gain Bandwidth (GBW) of 30 MHz while operating under a strict power budget of less than or equal to 300 uW per cell, this subsystem serves as a versatile analog front-end (AFE) baseline for modern communication and signal-processing chiplets.

---

## 3. Motivation & Technical Innovation
As wireless and wireline communication standards push into higher frequency bands, energy-efficient on-chip analog filtering becomes paramount. Traditional active-RC filter topologies are heavily constrained by the open-loop gain and bandwidth requirements of their operational amplifiers, making them power-hungry at multi-MHz frequencies.

This project implements a Gm-C realization of the classic KHN state-variable biquad architecture. By replacing conventional Op-Amps and area-intensive resistors with high-linearity transconductance (Gm) cells, the design shifts the poles entirely via current-charge dynamics onto integrating capacitors. The key innovations and advantages of this proposal include:
* **True State-Variable Versatility:** Simultaneous extraction of three filter characteristics (LP, BP, HP), reducing silicon area for multi-band systems.
* **Low Sensitivity:** Inherently low sensitivity to component variations compared to other biquad structures (like Sallen-Key).
* **Open-Source PDK Optimization:** Maximizing the headroom of the 3.3V thick-oxide primitives in the GF180MCU node to achieve robust linearity and a high signal-to-noise ratio (SNR) without exceeding strict power boundaries.

---

## 4. Subsystem Architectural Topology

The KHN biquad filter architecture consists of two cascading analog integrators and a summing/gain stage configured in a feedback loop.

<img width="1919" height="894" alt="image" src="https://github.com/user-attachments/assets/4b1fd5b1-bc02-4578-8930-a9e85db4a110" />

### A. The Core Gm Cell / Integration Element
To satisfy the 60 dB DC open-loop gain requirement while driving capacitive loads at high frequencies, a Folded Cascode OTA will be used as the Gm building block. The folded cascode structure provides the high output impedance necessary to achieve high DC voltage gain in a single stage, which keeps the phase errors in the integrator loop remarkably small.

### B. Tuning and Integration Network
The continuous-time integration is achieved by driving a balanced capacitor load (CL = 2 pF). This localized capacitance value is carefully optimized to balance the trade-offs between thermal noise (kT/C), silicon real estate, and achievable transconductance.

---

## 5. Design Specifications & Target Parameters

The core parameters for the individual Gm cells have been optimized for the robust 3.3V framework of the GF180MCU PDK:

| Target Parameter | Value | Design Context & Justification |
| :--- | :--- | :--- |
| **Process Node** | GF180MCU (180nm) | Standardized open-source foundry PDK. |
| **Supply Voltage (VDD)** | 3.3V | Maximizes output swing and linearity headroom. |
| **DC Open-Loop Gain (A0)** | >= 60 dB (1000 V/V) | Mitigates finite-gain errors to maintain sharp filter skirts. |
| **Gain Bandwidth (GBW)** | 30 MHz | Establishes the upper cutoff frequency boundary (f0). |
| **Phase Margin (PM)** | >= 60 degrees | Prevents phase lags that lead to unwanted Q-enhancement or oscillation in the loop. |
| **Slew Rate (SR)** | >= 20 V/us | Prevents large-signal distortion during high-frequency transients. |
| **Input Common Mode Range (ICMR-)** | >= 1.0V | Tailored for robust input pair biasing. |
| **Input Common Mode Range (ICMR+)** | >= 2.8V | Expanded upper bound to fully leverage the 3.3V rail. |
| **Integrator Load Capacitance (CL)** | 2 pF | Scaled to a realistic on-chip target for a 30 MHz cutoff footprint. |
| **Power Consumption (Pdiss)** | <= 300 uW per cell | Enforces strict power efficiency for multi-stage integration. |

---

## 6. Mathematical Framework

The nominal transconductance (Gm) value required for each integration block to achieve the target unity-gain frequency under the specified capacitive load is calculated as follows:

Gm = 2 * pi * GBW * CL = 2 * pi * 30 MHz * 2 pF = 377 uS

For the 2nd-order KHN filter response, the center frequency (w0) and Quality Factor (Q) are defined by the individual transconductances (Gm1, Gm2) and integration capacitors (C1, C2). Assuming identical matched blocks (Gm1 = Gm2 = Gm and C1 = C2 = C):

w0 = Gm / C

By sizing the scaling Gm elements in the feedback loop, the Q factor can be tuned independently of the center frequency (w0), ensuring excellent control over the passband shape.

---

## 7. Process Variation & Risk Mitigation Plan

Designing active filters in open-source PDKs without precise factory trimming networks introduces vulnerabilities to process corners:

* **Fast-Fast (FF) Corner:** Increases Gm, shifting the center frequency (f0) upward and potentially driving the system into instability if phase margins degrade.
* **Slow-Slow (SS) Corner:** Decreases Gm, shrinking the filter bandwidth and introducing unacceptable insertion loss.

### Physical Design Mitigation Strategies:
1. **Source Degeneration and Sizing:** Active resistors or passive poly-resistors will be integrated into the Gm cell source paths to linearize the transconductance stage, making it less sensitive to variations in Vth.
2. **Common-Centroid Layout Topology:** All matching differential pairs and critical current-mirror loads within the Gm blocks will be routed using a strict cross-coupled Common-Centroid layout design geometry. Dummy active regions will guard the arrays against processing variations and gradient errors across the die.

---

## 8. Open-Source EDA Toolchain & Verification Flow

The engineering design workflow will adhere completely to the open-source VLSI paradigm via WSL2/Ubuntu:

* **Schematic Capture (Xschem):** Designing the transistor-level topologies for the Gm cells, biasing networks, and the full state-variable feedback system using GF180MCU primitives.
* **Pre-Layout SPICE Verification (Ngspice):** Simulating AC responses (to verify LP, BP, HP frequency characteristics), DC sweeps, and transient parameters over complete PVT corners (TT, FF, SS) spanning -40°C to 125°C.
* **Layout Mask Design (Magic VLSI):** Manual routing of the custom analog layout, ensuring symmetric paths to maintain high Common-Mode Rejection Ratio (CMRR).
* **Physical Extraction and LVS (Netgen):** Running Layout Versus Schematic (LVS) checks to guarantee physical correctness before performing Parasitic Extraction (PEX) for final post-layout timing and frequency confirmation in Ngspice.

# Voltage-Biased Bandgap Reference & Two-Stage Miller OTA Design (GF180MCU)

This repository contains the project proposal, design topology, mathematical framework, and technical specifications for a high-performance analog subsystem consisting of a **Voltage-Mode Bandgap Reference (BGR)** and a **Two-Stage Miller Operational Transconductance Amplifier (OTA)**. 

The project is targeted for the open-source **GF180MCU 180nm CMOS technology** node using 3.3V primitives.

## Architectural Overview

Unlike traditional analog designs that utilize localized current mirrors to distribute bias, this architecture explores a unique **direct voltage-biasing paradigm**. The 1.23V output of the core Bandgap Reference is used to directly drive the gates of the OTA's NMOS current sinks ($M_5$ and $M_7$).

### Key Objectives
* **Maximize Transconductance ($g_m$):** Forcing current sinks into strong inversion.
* **Enhanced Unity Gain Bandwidth (UGB):** Optimized for high-speed signal processing.
* **Superior Slew Rate (SR):** Achieving $\geq 25\text{ V/}\mu\text{s}$ through fixed, high overdrive voltage dynamics.

---

## Circuit Topology & Design

The design is split into two primary functional blocks:

### 1. Two-Stage Miller OTA
* **Stage 1 (Gain Stage):** PMOS differential pair ($M_1, M_2$) driven by an NMOS tail current source ($M_5$). An active PMOS current mirror ($M_3, M_4$) performs differential-to-single-ended conversion.
* **Stage 2 (Output Stage):** PMOS common-source amplifier ($M_6$) driving an active NMOS current sink ($M_7$) to deliver a high output voltage swing.
* **Frequency Compensation:** A Miller capacitor ($C_c$) combined with a nulling resistor is implemented in the feedback loop to eliminate the Right-Half-Plane (RHP) zero and guarantee phase margin stability.

### 2. Voltage-Mode Bandgap Reference (BGR)
* Generates a stable, temperature-compensated $V_{BGR} \approx 1.23\text{V}$.
* Directly drives the heavy capacitive gate loads ($C_{gg5} + C_{gg7}$) without localized mirror buffering.

---

## Technical Specifications

### Two-Stage Miller OTA Targets
| Parameter | Direct Voltage-Bias Target | Design Impact & GF180MCU Context |
| :--- | :--- | :--- |
| **Supply Voltage ($V_{dd}$)** | 3.3 V | Constant DC supply ($V_{ss} = 0\text{V}$) |
| **DC Open-Loop Gain ($A_v$)** | 65 dB to 70 dB | Reduced due to lower channel-length modulation resistance ($r_o$) at high $V_{ov}$ |
| **Unity Gain Bandwidth (UGB)** | 25 MHz to 35 MHz | Increased via enhanced input transconductance ($g_{m1,2}$) |
| **Phase Margin ($\phi_m$)** | 55° to 60° | Strict RHP zero management; requires optimized sizing for $M_6$ |
| **Slew Rate (SR)** | $\geq 25\text{ V/}\mu\text{s}$ | Significantly higher due to fixed 1.23V bias delivering large tail currents |
| **Input Common Mode Range** | 1.3 V to 3.3 V | Compressed lower bound ($V_{in,min} = V_{ds,sat5} + V_{gs1}$) |
| **Output Voltage Swing** | 0.5 V to 3.1 V | Compressed lower bound limited by $V_{ds,sat7} \approx 0.48\text{V}$ |

### Bandgap Reference (BGR) Targets
* **Output Voltage ($V_{Bias}$):** 1.23 V (Fixed)
* **Temperature Coefficient:** $< 15\text{ ppm/}^\circ\text{C}$ (Critical due to exponential current dependencies on $V_{ov}$)
* **Power Supply Rejection (PSRR):** $\geq 60\text{ dB @ DC}$

---

## Process Variation & Risk Mitigation

Direct voltage-biasing lacks traditional process tracking. Since $V_g$ is clamped at a fixed 1.23V, variations in the threshold voltage ($V_{thn}$) pose corner-case risks:
* **Fast-Fast (FF) Corner:** $V_{thn}$ drops $\rightarrow$ $V_{ov}$ spikes $\rightarrow$ risk of excessive power dissipation/thermal runaway.
* **Slow-Slow (SS) Corner:** $V_{thn}$ rises $\rightarrow$ $V_{ov}$ collapses $\rightarrow$ risk of degraded $g_m$, UGB, and Slew Rate.

### Implemented Strategies:
1. **Geometric Mitigations:** Large channel lengths ($L \geq 2\,\mu\text{m}$) are mandated for $M_5$ and $M_7$ to improve output resistance, reduce Drain-Induced Barrier Lowering (DIBL), and minimize device mismatch.
2. **Layout Matching:** **Common-Centroid** cross-coupling topology layout strategy accompanied by dummy elements to protect against localized spatial gradients during fabrication.

---

## Mathematical Framework

### Overdrive Voltage & Current
With nominal $V_{thn} \approx 0.75\text{V}$ in the GF180MCU 3.3V process:
$$V_{ov} = V_{gs} - V_{thn} = 1.23\text{V} - 0.75\text{V} = 0.48\text{V} \, (480\text{ mV})$$

### Systematic Offset Mitigation Constraint
To maintain balance between the two stages and prevent systematic input offset voltage:
$$\frac{(W/L)_6}{(W/L)_{3,4}} = 2 \cdot \left[ \frac{(W/L)_7}{(W/L)_5} \right]$$

---

## 📁 Repository Structure
```text
├── design/              # Schematic entry files & testbenches
├── docs/                # Full Project Proposal & technical documentation
├── layout/              # GDSII/Layout files (Common-Centroid matched)
├── simulation/          # SPICE netlists and corner simulation logs (FF, SS, TT)
└── README.md            # Project Overview

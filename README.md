# Two-Stage CMOS Operational Amplifier (TSMC 180nm)

**Author:** Tapan Sanjeevkumar Gupta | AISSMS College of Engineering (SPPU)
**Project Context:** Analog Front-End Design / FOSSEE ECG 

This repository contains the complete design, hand calculations, and LTspice verification for a custom two-stage CMOS operational amplifier designed in a standard TSMC 180nm process. The architecture utilizes a differential input pair with an active current mirror load, followed by a common-source second stage and Miller pole-splitting compensation.

## 🎯 Design Specifications & Final Results

The amplifier was designed to balance high-speed transient response with strict low-power constraints. 

| Parameter | Target Specification | Achieved Result (ICMR+) | Achieved Result (ICMR-) |
| :--- | :--- | :--- | :--- |
| **Technology** | TSMC 180nm | TSMC 180nm | TSMC 180nm |
| **Supply Voltage ($V_{DD}$)** | 1.8 V | 1.8 V | 1.8 V |
| **DC Gain** | $\ge$ 60 dB (1000 V/V) | 70 dB | 68 dB |
| **Gain-Bandwidth (GBW)** | $\ge$ 30 MHz | 35 MHz | 31 MHz |
| **Phase Margin (PM)** | $\ge$ 60° | 61° | 65° |
| **Slew Rate (SR)** | $\ge$ 20 V/µs | 28 V/µs | 28 V/µs |
| **Load Capacitance ($C_L$)**| 2 pF | 2 pF | 2 pF |
| **Power Dissipation** | $\le$ 300 µW | < 300 µW | < 300 µW |

---

## 🧮 Design Methodology & Hand Calculations

The component sizing was executed through rigorous Phase-1 hand calculations before being mapped to the BSIM3 Level 49 SPICE models. 

### 1. Compensation & Slew Rate
* The Miller compensation capacitor ($C_c$) was calculated to satisfy $C_c \ge 0.22 C_L$ to secure a 60° Phase Margin. 
* To be safe, the capacitor was sized to 800 fF.
* Slew rate requirements dictated the tail current via the equation $I_5 = SR \times C_c$.
* This established a required tail current of 16 µA, which was rounded up to an operating bias of 20 µA.

### 2. First Stage (Differential Pair & Active Load)
* The required transconductance ($g_{m1}$) for a 30 MHz GBW was calculated as 160 µS.
* This resulted in an input pair sizing of $(W/L)_1 = 6.18$.
* To guarantee survival across the 0.8 V to 1.6 V Input Common-Mode Range (ICMR), the design compensated for the **Body Effect**.
* The maximum threshold voltage ($V_{T1(max)}$) was assumed to spike to 0.49 V at the ICMR+ limit. 
* The minimum threshold voltage ($V_{T1(min)}$) was assumed to be 0.3862 V.
* Sizing equations utilized these limits to yield an active load sizing of $(W/L)_3 = 9.5$ and a tail mirror sizing of $(W/L)_5 = 5.7$ to prevent the tail node from crashing into the triode region.

### 3. Second Stage (Common Source)
* To push the secondary pole past the 0 dB crossing, the transconductance of the second stage ($g_{m6}$) was targeted at $\ge 10 \times g_{m1}$.
* This required a massive $g_{m6} \ge 1600 \text{ \mu S}$.
* Initial calculations yielded $(W/L)_6 = 149$ and an NMOS sinker sizing of $(W/L)_7 = 45$, pulling an aggressive 157 µA of current.

---

## 🛠️ Simulation Tuning & Trade-offs

During LTspice `.op` and `.ac` verification, several physical non-idealities were encountered and mitigated:

### The Power Budget Crisis
* The initial second-stage mathematical sizing drew excessive current, causing the total power dissipation to violate the strict 300 µW budget.
* **Resolution:** The DC current was artificially throttled by pulling the width of the NMOS sinker down to 20 µm (from 22.5 µm). 
* **The High-Impedance Tug-of-War:** Starving the sinker caused the PMOS driver to fall out of saturation due to current mismatch.
* A rigorous tuning sweep was performed on the PMOS driver (74.5 µm $\to$ 63 µm $\to$ 58 µm $\to$ 128 µm $\to$ 63 µm) and the NMOS sinker (22.5 µm $\to$ 14.25 µm $\to$ 20 µm $\to$ 40 µm $\to$ 30 µm $\to$ 20 µm) until the high-impedance output node balanced perfectly, restoring deep saturation and voltage gain.

### Parasitic Penalties vs. Output Resistance
* An attempt was made to increase DC gain by doubling the channel lengths ($L = 1\text{ \mu m}$) to mitigate Channel Length Modulation.
* **Resolution:** While DC gain increased, the quadrupled gate capacitance choked the amplifier at the ICMR- boundary, degrading the GBW. The transistors were rolled back to the optimal 500 nm baseline, which successfully secured a robust $> 60\text{ dB}$ gain and 31 MHz GBW across the *entire* common-mode range.

---
*Verified via LTspice using TSMC 180nm foundry models.*

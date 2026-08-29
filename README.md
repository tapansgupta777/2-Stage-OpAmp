# Two-Stage CMOS Operational Amplifier (TSMC 180nm)

**Author:** Tapan Sanjeevkumar Gupta | AISSMS College of Engineering (SPPU)
**Project Context:** Analog IC Design / High-Speed Front-End

This repository contains the complete design, hand calculations, and LTspice verification for a custom two-stage CMOS operational amplifier designed in a standard TSMC 180nm process. The architecture utilizes a differential input pair with an active current mirror load, followed by a common-source second stage and Miller pole-splitting compensation.

## 🎯 Design Specifications & Final Results

The amplifier was designed to balance high-speed transient response with strict low-power constraints. 

| Parameter | Target Specification | Achieved Result (ICMR+) | Achieved Result (ICMR-) |
| :--- | :--- | :--- | :--- |
| **Technology** | TSMC 180nm | TSMC 180nm | TSMC 180nm |
| **Supply Voltage (VDD)** | 1.8 V | 1.8 V | 1.8 V |
| **DC Gain** | ≥ 60 dB (1000 V/V) | 70 dB | 68 dB |
| **Gain-Bandwidth (GBW)** | ≥ 30 MHz | 35 MHz | 31 MHz |
| **Phase Margin (PM)** | ≥ 60° | 61° | 65° |
| **Slew Rate (SR)** | ≥ 20 V/µs | 28 V/µs | 28 V/µs |
| **Load Capacitance (CL)**| 2 pF | 2 pF | 2 pF |
| **Power Dissipation** | ≤ 300 µW | < 300 µW | < 300 µW |

---

## 🧮 Comprehensive Hand Calculations

The component sizing was executed through rigorous Phase-1 hand calculations before being mapped to the BSIM3 Level 49 SPICE models. 

### 1. Compensation Capacitor ($C_c$) & Slew Rate (M5)
To secure a 60° Phase Margin, the secondary pole must be pushed past the 0 dB crossing. 

$$C_c \ge 0.22 C_L$$

$$C_c \ge 0.22 \times 2\text{ pF} = 440\text{ fF}$$

*   **Selected $C_c$:** 800 fF (To guarantee stability margin).

Slew rate requirements dictate the tail current:

$$I_5 = \text{SR} \times C_c$$

$$I_5 = (20\text{ V/u s}) \times 800\text{ fF} = 16\text{ u A}$$

*   **Selected Tail Current ($I_5$):** 20 µA.
*   **Branch Current ($I_D$):** 10 µA per branch.

### 2. Differential Input Pair (M1, M2)
The required transconductance ($g_{m1}$) for a 30 MHz GBW is:

$$g_{m1} = \text{GBW} \times C_c \times 2\pi$$

$$g_{m1} = 30\text{ MHz} \times 800\text{ fF} \times 2\pi \approx 150\text{ u S} \rightarrow \mathbf{160\text{ u S}}$$

Calculating the aspect ratio using $u_n C_{ox} = 207\text{ u A/V}^2$:

$$(\frac{W}{L})_{1,2} = \frac{g_{m1}^2}{u_n C_{ox} (2 I_D)}$$

$$(\frac{W}{L})_{1,2} = \frac{(160\text{ u S})^2}{207\text{ u A/V}^2 \times 20\text{ u A}} \approx \mathbf{6.18}$$

### 3. Active Load (M3, M4) & ICMR+
To guarantee survival at the 1.6 V Input Common-Mode Range (ICMR) maximum, the PMOS threshold voltage ($V_{T3}$) was extracted as 0.3906 V, and the NMOS threshold ($V_{T1}$) was assumed to shift to 0.3862 V due to the body effect.

$$(\frac{W}{L})_{3,4} = \frac{2 I_{D3}}{u_p C_{ox} [V_{DD} - \text{ICMR+} - |V_{T3}|_{max} + V_{T1(min)}]^2}$$

Using $u_p C_{ox} = 55\text{ u A/V}^2$:

$$(\frac{W}{L})_{3,4} = \mathbf{9.5}$$

### 4. Tail Current Mirror (M5) & ICMR-
To keep M5 out of the triode region at the 0.8 V ICMR limit, the worst-case maximum threshold voltage shift ($V_{T1(max)} = 0.49\text{ V}$) was utilized.

$$V_{DSAT5} \le \text{ICMR-} - V_{OV1} - V_{T1(max)}$$

$$V_{DSAT5} \le 0.8\text{ V} - 0.125\text{ V} - 0.49\text{ V} = \mathbf{184\text{ mV}}$$

$$(\frac{W}{L})_5 = \frac{2 I_{D5}}{u_n C_{ox} (V_{DSAT5})^2}$$

$$(\frac{W}{L})_5 = \mathbf{5.7}$$

### 5. Second Stage (M6, M7)
To ensure the second pole does not degrade the phase margin, $g_{m6}$ must be significantly larger than $g_{m1}$.

$$g_{m6} \ge 10 \times g_{m1} \rightarrow \mathbf{1600\text{ u S}}$$

Scaling from M4 to achieve this massive transconductance:

$$(\frac{W}{L})_6 = \mathbf{149}$$

$$(\frac{W}{L})_7 = \frac{I_7}{I_5} (\frac{W}{L})_5 = \mathbf{45}$$

---

## 🛠️ Simulation Tuning & Silicon Trade-offs

During LTspice `.op` and `.ac` verification with Level 49 models, physical non-idealities required strategic tuning of the calculated aspect ratios:

1. **The Power Budget Crisis:** The initial second-stage sizing ($W_7 = 45\text{ µm}$) drew 157 µA of current, causing the total chip power dissipation to violate the 300 µW budget.
2. **The High-Impedance Tug-of-War:** To save power, the NMOS sinker (M7) width was throttled down. This starved the PMOS driver (M6) and pushed it into the triode region, destroying the voltage gain.
3. **The Balance:** A rigorous tuning sweep was performed on M6 (74.5 µm $\to$ 63 µm $\to$ 128 µm $\to$ 63 µm) and M7 (22.5 µm $\to$ 14.25 µm $\to$ 40 µm $\to$ 20 µm) until the output node balanced perfectly, restoring deep saturation and pushing the DC gain back to $\ge 68\text{ dB}$.
4. **Parasitic Capacitance Limits:** Channel lengths ($L$) were tested at 1 µm to boost output resistance, but the quadrupled gate capacitance choked the GBW at the ICMR- boundary. The design was rolled back to the 500 nm baseline, which successfully secured the 30 MHz GBW across all process corners. 

---
*Verified via LTspice using TSMC 180nm foundry models.*

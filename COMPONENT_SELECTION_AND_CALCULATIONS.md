# Component Selection & Engineering Calculations
### Thermal Estimates, Inductor Sizing, and Footprint Justifications

**Author:** Ajith Shajan  
**Degree:** B.Tech in Electronics & Communication Engineering  
**Institution:** Govt. Model Engineering College, Kochi (KTU)  
**Project:** Automotive BLE-to-CAN Hardware Gateway  
**Toolchain:** KiCad 10.0 (Custom footprint library in `footprints.pretty/`)  

---

## 1. Core Integrated Circuits (U1–U7)

| Ref | IC Part Number | Function | Package Code | Assigned Footprint | Technical Selection Rationale |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **U1** | `LM5164DDA` | 100 V, 1 A Synchronous Buck (Main 3.3 V) | HSOP-8-1EP (DDA) | `Package_SO:HSOP-8-1EP_3.9x4.9mm_P1.27mm_EP2.41x3.1mm_ThermalVias` | 100 V wide-$V_{IN}$ automotive regulator. Exposed thermal pad ($2.41 \times 3.1\text{ mm}$) with thermal vias directly tied to GND plane is required to keep junction temperature below $125^\circ\text{C}$ during sustained $36\text{ V}$ operation. |
| **U2** | `CC2340R52E0RKPR` | 2.4 GHz BLE 5.3 SoC (TI SimpleLink) | VQFN-40 (RKP) | `footprints:RKP0040B-MFG` | $5.0 \times 5.0\text{ mm}$, $0.4\text{ mm}$ pitch. Center exposed ground pad provides low-impedance RF ground return and thermal relief. Official TI manufacturer footprint exported via UltraLibrarian. |
| **U3** | `TCAN4550RGY` | Automotive CAN FD Controller + Transceiver | VQFN-20 (RGY) | `Package_DFN_QFN:Texas_RGY_R-PVQFN-N20_EP2.05x3.05mm` | $3.5 \times 4.5\text{ mm}$, $0.5\text{ mm}$ pitch. The integrated CAN FD transceiver generates heat during high-speed $5\text{ Mbps}$ bus arbitration; the $2.05 \times 3.05\text{ mm}$ exposed thermal pad is soldered to Layer 2 ground. |
| **U4** | `LMR36506MSC5RPERQ1` | 65 V, 2.2 MHz Synchronous Buck (5 V Charger) | VQFN-HR-9 (RPE) | `footprints:RPE0009A` | $2.0 \times 2.0\text{ mm}$ HotRod™ package. Solder-bumped flip-chip die eliminates bond wire parasitic inductance, critical for $2.2\text{ MHz}$ operation to satisfy CISPR 25 Class 5 EMI without large external filters. |
| **U5** | `BQ24040DSQR` | Li-ion Linear Charger (250 mA, Auto-Power) | WSON-10 (DSQ) | `footprints:DSQ10_P9X1P5` | $2.0 \times 2.0\text{ mm}$, $0.4\text{ mm}$ pitch, $0.9 \times 1.5\text{ mm}$ exposed pad. Dissipates $(5.0\text{ V} - 3.2\text{ V}) \times 0.25\text{ A} = 0.45\text{ W}$ during deeply discharged pre-charge; thermal pad prevents thermal foldback. |
| **U6** | `TPS63900DSKR` | High-Efficiency Buck-Boost (3.3 V Battery Rail) | WSON-10 (DSK) | `Package_SON:WSON-10-1EP_2.5x2.5mm_P0.5mm_EP1.2x2mm_ThermalVias` | $2.5 \times 2.5\text{ mm}$, $0.5\text{ mm}$ pitch, $1.2 \times 2.0\text{ mm}$ thermal pad. Handles smooth transition between buck and boost modes as single-cell Li-ion discharges from $4.2\text{ V}$ down to $3.0\text{ V}$. |
| **U7** | `PTPS2116DRLR` | Low-Iq Power MUX (Seamless Auto-Switchover) | SOT-583 (DRL) | `footprints:DRL0008A-MFG` | $1.6 \times 1.2\text{ mm}$, $0.5\text{ mm}$ pitch. Sub-microamp quiescent current power switch that automatically transfers power between vehicle regulator ($3V3\_REG$) and backup battery ($3V3\_BAT$). |

---

## 2. Power & Protection Silicon (Q1, F1, D1, D2, D3, D5)

| Ref | Schematic Part | Recommended Automotive Part | Assigned Footprint | Technical Selection Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **Q1** | `Q_PMOS_GSD` | **Diodes Inc DMP4065S-7** (AEC-Q101, 40 V, 2.4 A) | `Package_TO_SOT_SMD:SOT-23` | **Pinout Verification:** DMP4065S-7 SOT-23 pinout is Pin 1 = Gate, Pin 2 = Source, Pin 3 = Drain. This **exactly matches** the placed KiCad `Q_PMOS_GSD` symbol. SOT-23 handles up to $2.4\text{ A}$ continuous ($R_{DS(on)} < 65\text{ m}\Omega$), perfectly sized for the $1\text{ A}$ max system consumption with minimal board area. |
| **F1** | `Fuse` | **Littelfuse 0451002.MRL** / **Schurter UST 1206** | `Fuse:Fuse_1206_3216Metric` | $2\text{ A}$ slow-blow, $65\text{ VDC}$ automotive rating. 1206 ceramic body withstands inrush currents from input MLCC charging while providing guaranteed interruption under catastrophic short circuit. |
| **D1** | `D_TVS` | **Vishay TPSMC36A-E3/57T** (AEC-Q101) | `Diode_SMD:D_SMC` | $1500\text{ W}$ peak pulse power, $V_{RWM} = 33\text{ V}$, $V_{BR} = 36.7\text{ V}$. Sized in SMC (DO-214AB) to survive ISO 7637-2 Pulse 2a ($+50\text{ V}$) and Pulse 3a/3b ($\pm 150\text{ V}$) inductive transients without degradation. |
| **D2** | `D_Zener` | **Nexperia BZX384-C15** / **MMSZ5245BT1G** | `Diode_SMD:D_SOD-123` | $15\text{ V}$ clamping zener across P-MOSFET gate-to-source. Protects $V_{GS}$ ($V_{GS(max)} = \pm 20\text{ V}$) from exceeding breakdown when $V_{IN}$ rises to $36\text{ V}$. SOD-123 offers robust automotive wave/reflow solderability. |
| **D3** | `D_TVS` | **Infineon ESD112-B1-02ELS** / **PESD5V0X1BCSF** | `Diode_SMD:D_0402_1005Metric` | Bidirectional RF ESD protection ($C_D < 0.3\text{ pF}$, insertion loss $< 0.1\text{ dB}$ at $2.4\text{ GHz}$). Enclosed in 0402 footprint placed directly at antenna feed pad to absorb ISO 10605 $\pm 8\text{ kV}$ contact discharge without detuning the RF match. |
| **D5** | `D_TVS_Dual_AAC` | **Nexperia PESD2CAN** / **ON Semi NUP2105L** | `Package_TO_SOT_SMD:SOT-23` | Dual bidirectional CAN bus ESD suppressor ($24\text{ V}$ standoff, $C_{io} < 30\text{ pF}$). SOT-23 package allows symmetrical routing between CANH and CANL to minimize signal skew and common-mode radiation. |

---

## 3. Power Inductors, RF Coils & CAN Choke (L1–L6, FL1)

| Ref | Nominal Value | Function | Selected Part Number | Assigned Footprint | Detailed Design & Calculation Verification |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **L1** | $47\text{ }\mu\text{H}$ | LM5164 Main Buck Inductor | **Bourns SRN6045TA-470M** (AEC-Q200) | `Inductor_SMD:L_Bourns_SRN6045TA` | **Operating Point Validation:** With $R_{ON} = 100\text{ k}\Omega$ ($R_2$), switching frequency $f_{SW} \approx 300\text{ kHz}$.<br>Ripple current: $\Delta I_L = \frac{V_{OUT}}{f_{SW} \cdot L} (1 - \frac{V_{OUT}}{V_{IN}})$.<br>At $V_{IN} = 12\text{ V}$: $\Delta I_L = \frac{3.3}{300\text{k} \cdot 47\mu} (1 - \frac{3.3}{12}) = 0.40\text{ A}$ ($40\%$ of $1\text{ A}$ rating).<br>At $V_{IN} = 24\text{ V}$: $\Delta I_L = 0.50\text{ A}$.<br>Bourns SRN6045TA provides $1.6\text{ A}$ RMS, $2.0\text{ A}$ $I_{sat}$, shielded $6.0 \times 6.0\text{ mm}$ package, preventing magnetic coupling to adjacent analog nets. |
| **L2** | $10\text{ }\mu\text{H}$ | CC2340 Internal DCDC Buck Inductor | **Murata DFE201610E-100M** | `Inductor_SMD:L_0805_2012Metric` | $2.0 \times 1.6\text{ mm}$ ($0805$) metal alloy core, $I_{sat} = 650\text{ mA}$, low DCR. Specifically approved in TI CC2340 reference design for internal core DC/DC converter. |
| **L3** | $2.0\text{ nH}$ | CC2340 RF Differential-to-Single Match | **Murata LQW15AN2N0B** / **Coilcraft 0402HP** | `Inductor_SMD:L_0402_1005Metric` | High-Q ($Q > 30$ at $2.4\text{ GHz}$) wirewound chip inductor. 0402 metric package is strictly required to prevent self-resonance and uncontrolled parasitic inductance. |
| **L4** | $2.8\text{ nH}$ | CC2340 RF Harmonic Filter | **Murata LQW15AN2N8B** / **Coilcraft 0402HP** | `Inductor_SMD:L_0402_1005Metric` | High-Q wirewound chip inductor. 0402 package matches TI reference layout to suppress 2nd ($4.8\text{ GHz}$) and 3rd ($7.2\text{ GHz}$) harmonics below regulatory limits. |
| **L5** | $6.8\text{ }\mu\text{H}$ | LMR36506 5 V Buck Inductor | **Coilcraft XFL4020-682ME** (AEC-Q200) | `Inductor_SMD:L_Coilcraft_XFL4020` | TI LMR36506-Q1 datasheet Table 9-2 explicitly recommends $6.8\text{ }\mu\text{H}$ for $V_{OUT} = 5\text{ V}$ at $2.2\text{ MHz}$. Coilcraft XFL4020 ($4.0 \times 4.0 \times 2.1\text{ mm}$) provides $I_{sat} = 1.6\text{ A}$, low AC core loss, and magnetic shielding for CISPR 25 Class 5. |
| **L6** | $2.2\text{ }\mu\text{H}$ | TPS63900 Buck-Boost Inductor | **Coilcraft LPS3015-222ML** (AEC-Q200) | `Inductor_SMD:L_Coilcraft_LPS3015` | TI TPS63900 specifies $2.2\text{ }\mu\text{H}$ for optimal transient stability and ultra-low quiescent current. LPS3015 ($3.0 \times 3.0 \times 1.5\text{ mm}$) has $I_{sat} = 2.0\text{ A}$ (at 10% drop), ensuring zero saturation during load steps up to $400\text{ mA}$. |
| **FL1**| $100\text{ }\mu\text{H}$ | CAN Bus Common-Mode Choke | **Murata DLW43SH101XK2** (AEC-Q200) | `Inductor_SMD:L_CommonModeChoke_Coilcraft_1812CAN` | **Footprint & Pinout Compatibility:** EIA 1812 ($4.5 \times 3.2\text{ mm}$) 4-pad standard layout. Pad 1 & Pad 2 = Coil 1 (CANH); Pad 4 & Pad 3 = Coil 2 (CANL). Suppresses CAN FD common-mode noise between $10\text{ MHz}$ and $500\text{ MHz}$ without attenuating differential signaling. |

---

## 4. Crystals & Antenna (Y1, Y2, Y3, AE1)

| Ref | Frequency / Type | Critical Specifications | Selected Part Number | Assigned Footprint | Technical Selection Rationale |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Y1** | $48.000\text{ MHz}$ | CC2340 Primary High-Speed Clock | **NDK NX2016SA-48M** ($C_L = 8\text{ pF}$, $\pm 10\text{ ppm}$) | `Crystal:Crystal_SMD_2016-4Pin_2.0x1.6mm` | 4-pad miniature ceramic SMD ($2.0 \times 1.6\text{ mm}$). Grounding diagonal pads (P2, P4) provides Faraday shielding to minimize phase noise and frequency drift across automotive temperature ($-40^\circ\text{C}$ to $+105^\circ\text{C}$). |
| **Y2** | $32.768\text{ kHz}$ | RTC / BLE Sleep Clock | **Seiko Epson FC-135** ($C_L = 9\text{ pF}$, $\pm 20\text{ ppm}$) | `Crystal:Crystal_SMD_3215-2Pin_3.2x1.5mm` | 2-pad tuning-fork SMD ($3.2 \times 1.5\text{ mm}$). Ultra-low power clock enabling the CC2340 to remain in sub-microamp standby mode between BLE advertising/connection events. |
| **Y3** | $40.000\text{ MHz}$ | TCAN4550 CAN Controller Clock | **NDK NX2016SA-40M-STD-CZS-3** ($C_L = 8\text{ pF}$) | `Crystal:Crystal_SMD_2016-4Pin_2.0x1.6mm` | 4-pad SMD ($2.0 \times 1.6\text{ mm}$). Official reference crystal specified in TI TCAN4550-Q1 EVM. Extremely tight frequency tolerance ($\pm 20\text{ ppm}$) required for error-free $5\text{ Mbps}$ CAN FD bit timing synchronization. |
| **AE1**| $2.4\text{ GHz}$ | BLE Chip Antenna | **Johanson 2450AT18D0100** | `RF_Antenna:Johanson_2450AT18x100` | EIA 1206 ($3.2 \times 1.6\text{ mm}$) ceramic chip antenna. Matches symbol `Device:Antenna_Chip` with Pad 1 = Feed and Pad 2 = Ground/Trace return. Designed with specific copper keepout under antenna footprint. |

---

## 5. Capacitor Sizing & DC-Bias Derating Matrix (C1–C46)

### DC-Bias Derating Background
Class II ceramic dielectric capacitors (X7R, X5R) lose effective capacitance when exposed to continuous DC voltage. In automotive designs:
* A $10\text{ }\mu\text{F}$ capacitor rated for $16\text{ V}$ in an **0603** package drops by **up to 70%** under a $12\text{ V}$ bias, leaving only $\sim 3\text{ }\mu\text{F}$ effective capacitance.
* By upgrading bulk capacitors on higher-voltage and sensitive rails to **0805** or **1210**, effective capacitance remains within the regulator stability boundaries.

| Group | Capacitors | Nominal Value & Function | Rail Voltage | Voltage Rating | Assigned Footprint | Justification & Derating Safety Analysis |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **High-Voltage Automotive Bulk** | `C29` | $10\text{ }\mu\text{F}$ (TCAN4550 VSUP) | $9\text{--}36\text{ V}$ | $50\text{ V}$ | `Capacitor_SMD:C_1210_3225Metric` | Operating at $14\text{ V}$ nominal and $36\text{ V}$ max. A 1210 package ($3.2 \times 2.5\text{ mm}$) maintains $\ge 6.5\text{ }\mu\text{F}$ effective capacitance under full $36\text{ V}$ bias, ensuring transceiver transient supply stability. |
| | `C36` | $2.2\text{ }\mu\text{F}$ (LMR36506 VIN Input) | $9\text{--}36\text{ V}$ | $50\text{ V}$ | `Capacitor_SMD:C_0805_2012Metric` | High-frequency input switching loop capacitor for $2.2\text{ MHz}$ buck. 0805 provides low ESL/ESR while sustaining $50\text{ V}$ rating. |
| **Regulator Bulk Output MLCCs** | `C26` | $10\text{ }\mu\text{F}$ (TCAN4550 VCCOUT) | $5.0\text{ V}$ | $16\text{ V}$ | `Capacitor_SMD:C_0805_2012Metric` | Internal 5 V LDO output. 0805 maintains $> 7.5\text{ }\mu\text{F}$ effective capacitance under 5 V bias, meeting TI datasheet stability window ($4.7\text{--}47\text{ }\mu\text{F}$). |
| | `C35` | $22\text{ }\mu\text{F}$ (LMR36506 5 V VOUT) | $5.0\text{ V}$ | $16\text{ V}$ | `Capacitor_SMD:C_0805_2012Metric` | Buck output bulk capacitor. 0805 X7R provides low ripple voltage without excessive footprint area. |
| | `C38, C39` | $4.7\text{ }\mu\text{F} \times 2$ (TPS63900 VOUT) | $3.3\text{ V}$ | $10\text{ V}$ | `Capacitor_SMD:C_0805_2012Metric` | Dual parallel output caps ($9.4\text{ }\mu\text{F}$ nominal) maintain $> 7\text{ }\mu\text{F}$ effective under $3.3\text{ V}$ bias for buck-boost loop compensation. |
| | `C40` | $22\text{ }\mu\text{F}$ (TPS63900 VIN / BAT+) | $3.0\text{--}4.2\text{ V}$ | $10\text{ V}$ | `Capacitor_SMD:C_0805_2012Metric` | Li-ion battery input decoupling. Low ESR prevents battery voltage sag during heavy peak RF transmissions. |
| | `C41` | $10\text{ }\mu\text{F}$ (TPS2116 VIN1 / 3V3_REG) | $3.3\text{ V}$ | $10\text{ V}$ | `Capacitor_SMD:C_0805_2012Metric` | Absorbs line transient step during power MUX switchover from vehicle supply. |
| | `C44` | $10\text{ }\mu\text{F}$ (TPS2116 VOUT / +3.3V) | $3.3\text{ V}$ | $10\text{ V}$ | `Capacitor_SMD:C_0805_2012Metric` | Master $+3.3\text{ V}$ system rail output reservoir. Guarantees monotonic rail hold-up during MUX break-before-make transition. |
| | `C1` | $10\text{ }\mu\text{F}$ (LM5164 Input Bulk) | $9\text{--}36\text{ V}$ | $50\text{ V}$ | `Capacitor_SMD:C_0603_1608Metric` | Input high-frequency bypass for LM5164. |
| | `C3` | $22\text{ }\mu\text{F}$ (LM5164 Output Bulk) | $3.3\text{ V}$ | $10\text{ V}$ | `Capacitor_SMD:C_0603_1608Metric` | Main 3.3 V buck output reservoir. |
| | `C12` | $10\text{ }\mu\text{F}$ (CC2340 VDD Core Bulk) | $3.3\text{ V}$ | $10\text{ V}$ | `Capacitor_SMD:C_0603_1608Metric` | MCU power domain bulk decoupling. |
| **RF Matching & Clock Load Caps** | `C14, C15` | $12\text{ pF}, 10\text{ pF}$ (Y1 48 MHz Load) | N/A | $50\text{ V}$ | `Capacitor_SMD:C_0402_1005Metric` | 0402 C0G/NP0 dielectric ($0\pm 30\text{ ppm/}^\circ\text{C}$). Placed immediately at crystal pads for tight frequency stability. |
| | `C17–C23` | $1.2\text{--}15\text{ pF}$ (2.4 GHz RF Match) | N/A | $50\text{ V}$ | `Capacitor_SMD:C_0402_1005Metric` | 0402 C0G/NP0 high-Q RF capacitors. Small footprint minimizes trace discontinuities on the $50\text{ }\Omega$ coplanar waveguide. |
| | `C27, C28` | $8.2\text{ pF}$ (Y3 40 MHz Load) | N/A | $50\text{ V}$ | `Capacitor_SMD:C_0402_1005Metric` | 0402 C0G/NP0 load capacitors matching NDK $8\text{ pF}$ crystal load spec ($C_{load} = 2(C_L - C_{stray})$). |
| **Standard Decoupling & Filtering** | `C2, C4, C5–C11, C13, C16, C24, C25, C30–C34, C37, C42, C43, C45, C46` | $100\text{ nF}, 1\text{ }\mu\text{F}, 330\text{ nF}, 4.7\text{ nF}$ | Varied | $16\text{--}50\text{ V}$ | `Capacitor_SMD:C_0603_1608Metric` | Standard 0603 automotive X7R decoupling capacitors. Standardizing on 0603 for small-signal bypass simplifies pick-and-place and optimizes assembly cost. |

---

## 6. Resistor Sizing & Precision Matrix (R1–R25)

| Reference | Value | Function | Power / Tolerance | Assigned Footprint | Technical Selection Rationale |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R23** | $110\text{ k}\Omega$ | CC2340 `RBIAS` Reference (Pin 38) | $0.1\%$, $1/16\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | Precision internal current bias reference. Placed close to Pin 38 to minimize noise pickup. |
| **R11, R12** | $60.4\text{ }\Omega$ | CAN Bus Split Termination | $1\%$, $1/8\text{ W}$ | `Resistor_SMD:R_0805_2012Metric` | Sized in **0805** to safely withstand continuous automotive bus short-to-battery conditions ($36\text{ V}$) and ISO 7637-3 fast transient pulses across the $120\text{ }\Omega$ total termination. |
| **R1, R2** | R, $100\text{ k}\Omega$ | LM5164 EN Divider & $R_{ON}$ | $1\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | $R_2 = 100\text{ k}\Omega$ programs $300\text{ kHz}$ switching frequency. Standard 0603 provides high stability. |
| **R3, R4** | R, R | LM5164 Feedback Divider | $1\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | Low-tolerance divider setting the regulated $3V3\_REG$ output. |
| **R5** | $100\text{ k}\Omega$ | Q1 P-MOSFET Gate-to-Source Pullup | $5\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | Holds P-FET off during reverse polarity; limits current through Zener D2 during forward operation. |
| **R6** | R | LM5164 Bias Setting | $1\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | Standard bias resistor. |
| **R7** | $100\text{ k}\Omega$ | CC2340 Reset Pullup (`RST_N`) | $5\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | Digital pullup to $+3.3\text{ V}$. Combined with C46 ($100\text{ nF}$) forms a $10\text{ ms}$ reset debounce filter. |
| **R8** | $10\text{ k}\Omega$ | TCAN4550 Reset Pulldown (`RST`) | $5\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | TCAN4550 reset is active-high; pulldown prevents false resets during MCU boot. |
| **R9, R10** | $10\text{ k}\Omega$ | TCAN4550 `nWKRQ` & `nINT` Pullups | $5\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | Open-drain digital output pullup resistors to $+3.3\text{ V}$. |
| **R13, R14** | $1\text{ M}\Omega, 1\text{ M}\Omega$ | Battery Voltage Divider (`VBAT_SENSE`) | $0.5\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | Ultra-high impedance divider divides $4.2\text{ V}$ down to $2.1\text{ V}$ for MCU ADC with only $2.1\text{ }\mu\text{A}$ standby leakage. |
| **R15, R16** | $2.16\text{ k}\Omega, 2\text{ k}\Omega$ | BQ24040 Charge Current (`ISET`, `ITERM`) | $1\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | $R_{ISET} = 2.16\text{ k}\Omega$ programs $I_{CHG} = 250\text{ mA}$; $R_{ITERM} = 2\text{ k}\Omega$ sets termination threshold to $25\text{ mA}$ ($10\%$). |
| **R17–R21** | $36.5\text{ k}, 511, 9.53\text{ k}, 5\text{ k}$ | TPS63900 `VSEL` & Configuration | $1\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | Logic programming resistors configuring buck-boost output to fixed $3.3\text{ V}$. |
| **R22** | $1\text{ k}\Omega$ | 3.3 V Power Status LED Resistor | $5\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | Sets green LED forward current to $\sim 1.3\text{ mA}$ for clear indication with minimal standby drain. |
| **R24, R25** | $10\text{ k}\Omega, 1\text{ k}\Omega$ | SWD Debug Pullup / Series Resistors | $5\%$, $1/10\text{ W}$ | `Resistor_SMD:R_0603_1608Metric` | Signal integrity and line protection for SWD programming interface. |

---

## 7. Connectors, Test Points & Indicators

| Ref | Function | Hardware Part Number | Assigned Footprint | Technical Selection Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **J1** | Main Vehicle Power Input | 3-pin $2.54\text{ mm}$ Header | `Connector_PinHeader_2.54mm:PinHeader_1x03_P2.54mm_Vertical` | Lab prototype pin header (Pin 1: VIN, Pin 2: GND, Pin 3: IGN/Wake). Standard $0.1"$ pitch supports laboratory bench testing. |
| **J2** | Automotive CAN Bus Port | 3-pin $2.54\text{ mm}$ Header | `Connector_PinHeader_2.54mm:PinHeader_1x03_P2.54mm_Vertical` | Pin 1: CANH, Pin 2: CANL, Pin 3: GND. Allows easy interface to CANoe/CANalyzer or vehicle OBD-II tap. |
| **JP1**| CAN $120\text{ }\Omega$ Split Termination | 2-pin $2.54\text{ mm}$ Jumper | `Connector_PinHeader_2.54mm:PinHeader_1x02_P2.54mm_Vertical` | Jumper link enables connecting or disconnecting the onboard $120\text{ }\Omega$ bus termination depending on whether the node sits at a bus endpoint or intermediate stub. |
| **J3** | CC2340 SWD Debug Header | 5-pin $2.54\text{ mm}$ Header | `Connector_PinHeader_2.54mm:PinHeader_1x05_P2.54mm_Vertical` | Standard 5-pin ARM SWD pinout (Pin 1: VTREF $+3.3\text{ V}$, Pin 2: SWDIO, Pin 3: SWCLK, Pin 4: MCU_RESET, Pin 5: GND) compatible with J-Link and TI XDS110 debug probes. |
| **J_BAT1**| Single-Cell Li-ion Battery Header | **JST B3B-PH-K** ($2.0\text{ mm}$ TH) | `Connector_JST:JST_PH_B3B-PH-K_1x03_P2.00mm_Vertical` | Polarized, friction-locking 3-pin connector (Pin 1: BAT+, Pin 2: NTC Thermistor, Pin 3: GND). Through-hole pins resist mechanical pull stress during battery harness installation. |
| **TP1–TP8**| Hardware Bring-Up Test Points | $1.5\text{ mm}$ SMT Test Pad | `TestPoint:TestPoint_Pad_D1.5mm` | Placed on critical power and communication rails (`VIN_PROT`, `3V3_REG`, `5V_CHG`, `VBAT`, `3V3_BAT`, `+3.3V`, `CANH`, `CANL`) for oscilloscope probing and automated pogo-pin test fixtures. |
| **D4** | $+3.3\text{ V}$ Power OK Status Indicator | Green 0603 SMD LED | `LED_SMD:LED_0603_1608Metric` | High-efficiency 0603 green LED for visual bring-up confirmation. |

---

## 8. DFMEA & High-Risk Failure Mode Mitigations

| Failure Mode | Root Cause | Engineering Design Mitigation | Footprint / Layout Strategy |
| :--- | :--- | :--- | :--- |
| **Vehicle Battery Inversion** | Operator connects battery backwards ($-12\text{ V} / -24\text{ V}$). | P-MOSFET Q1 reverse polarity gate circuit with D2 Zener clamp. | SOT-23 package with verified GSD pinout ensures Q1 stays off, blocking current without thermal runaway. |
| **CAN Bus Standoff Overvoltage** | CANH/CANL wires shorted to $+24\text{ V}$ / $+36\text{ V}$ truck battery rail. | TCAN4550-Q1 transceiver handles continuous $\pm 58\text{ V}$ bus fault voltage. | Termination resistors R11/R12 in **0805** package to withstand prolonged $36\text{ V}$ dissipation ($V^2/R$). |
| **Buck Converter Thermal Overload** | Continuous 1 A load from 36 V input creates high switching and conduction loss. | LM5164 synchronous rectification (integrated high-side and low-side FETs). | HSOP-8 PowerPAD with dedicated thermal vias into solid internal Ground Layer 2. |
| **Capacitor DC-Bias Starvation** | MLCC loses 70% capacitance under continuous 36 V vehicle input. | Dielectric volume increased on all critical power nets. | Sized input bulk caps in **1210** (`C29`) and **0805** (`C36`), maintaining effective capacitance $> 6.5\text{ }\mu\text{F}$. |
| **RF Detuning & Radiation Loss** | PCB copper traces underneath chip antenna shift resonant frequency. | Johanson 2450AT18 series ground clearance. | Ground copper keepout enforced across all layers directly below `AE1` radiating element. |

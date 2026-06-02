# ⚡ PWM DC Motor Speed Controller

A high-reliability, discrete-component PWM motor speed controller built around the **LM555CM timer** and **IRFZ44N power MOSFET**. Designed with professional-grade component derating, multi-stage filtering, and dual-rail power isolation — outperforming most commercial PWM modules in real-world durability.

---

## 📷 Schematic

![PWM Motor Controller Schematic](schematic.jpg)

---

## ✨ Features

- Variable speed control via potentiometer (0–100% duty cycle)
- Dual power rail — noisy 20V motor rail fully isolated from sensitive 555 logic
- Triple-barrier noise filtering on control supply
- Active MOSFET cooling via BLDC fan
- Heavily derated components for long-term reliability
- Back-EMF protection on motor and filter inductor
- No gate driver IC required — 555 push-pull output drives MOSFET directly

---

## 🔧 Specifications

| Parameter | Value |
|---|---|
| Input Voltage | 20V DC |
| Logic Rail | 12V (LM7812CT regulated) |
| PWM Method | 555 Timer Astable |
| Switching Device | IRFZ44N N-Channel MOSFET |
| Max Continuous Current | ~10A (limited by D1) |
| Max Current (D1 upgraded) | ~20–25A with active cooling |
| Flyback Diode Rating | 40A TO-220 Schottky |
| Cooling | Active BLDC fan |
| Speed Control | Potentiometer (R1 100kΩ) |

---

## 🧩 Bill of Materials

| Reference | Component | Value / Part | Notes |
|---|---|---|---|
| U1 | Voltage Regulator | LM7812CT | 12V logic rail supply |
| U2 | Timer IC | LM555CM | PWM signal generation |
| Q1 | N-Channel MOSFET | IRFZ44N | Motor switching, TO-220 |
| D1 | Rectifier Diode | 1N1199C | Input protection ~12A |
| D2 | Schottky Diode | 1N5827 | L1 freewheeling |
| D3 | Rectifier Diode | 1N4007GP | Regulator output protection |
| D4 | Signal Diode | 1N4148 | Regulator circuit |
| D5, D6 | Signal Diode | 1N4148 | Asymmetric PWM duty cycle |
| D7 | Schottky Diode | 40A TO-220 | Motor flyback protection |
| L1 | Inductor | 4.3µH | Motor rail noise filter |
| C1, C2 | Electrolytic Cap | 1000µF | Input bulk filtering |
| C3 | Film Cap | 1µF | Input HF bypass |
| C4 | Ceramic Cap | 100nF | Regulator output HF bypass |
| C5 | Electrolytic Cap | 10µF | Regulator output decoupling |
| C6 | Film Cap | 1µF | Regulator output mid-freq |
| C7 | Electrolytic Cap | 1000µF | 555 VCC bulk supply |
| C8 | Ceramic Cap | 100nF | 555 VCC HF bypass |
| C9 | Ceramic Cap | 10nF | 555 timing capacitor |
| C10 | Ceramic Cap | 100nF | 555 pin 5 bypass |
| R1 | Potentiometer | 100kΩ | Speed / duty cycle control |
| R2 | Resistor | 1kΩ | 555 timing network |
| R3 | Resistor | 1kΩ | Series gate resistor |
| R4 | Resistor | 10kΩ | Gate-to-source pull-down |
| — | BLDC Fan | 12V | Active MOSFET cooling |

---

## 🏗️ Circuit Architecture

### Dual Power Rail Design

```
20V DC Input
    │
    ├──► Motor Rail (20V) ──► L1 filter ──► Motor + MOSFET switch
    │         │
    │      C1, C2, C3 bulk + HF filtering
    │
    └──► LM7812CT ──► 12V Logic Rail ──► 555 Timer PWM circuit
              │
         C7, C8, C9, C10 multi-stage filtering
```

The motor and logic rails are completely separated. The LM7812CT provides ~60dB Power Supply Rejection Ratio (PSRR), attenuating motor switching noise by 1000× before it reaches the 555 timer.

### PWM Generation (555 Astable)

```
12V ──► R2 ──┬──► D5 ──► R1 (pot) ──► D6 ──┬──► C9 ──► GND
             │                               │
           Pin 7 (DIS)                   Pin 2/6 (TRI/THR)
```

D5 and D6 route charge and discharge through opposite ends of the potentiometer, allowing duty cycle adjustment without affecting frequency — a classic variable PWM technique.

### Gate Drive

```
555 Pin 3 (push-pull) ──► R3 (1kΩ) ──► IRFZ44N Gate
                                              │
                                         R4 (10kΩ)
                                              │
                                             GND
```

- **R3** damps gate ringing and limits inrush current
- **R4** ensures gate is pulled firmly to ground when 555 output is low
- 555 push-pull output actively drives gate both high and low

---

## 🔍 How It Works

1. **555 Timer** generates a variable duty cycle PWM square wave on pin 3
2. **PWM signal** drives the gate of the IRFZ44N through R3
3. **IRFZ44N** switches the motor between 20V and ground at PWM frequency
4. **Average motor voltage** = Supply × Duty Cycle → controls motor speed
5. **D7** freewheels motor inductive current during MOSFET off-time
6. **BLDC fan** keeps MOSFET junction temperature low under load

---

## 🛡️ Why This Is More Reliable Than Commercial Modules

| Aspect | This Circuit | Typical Commercial Module |
|---|---|---|
| Flyback diode | 40A TO-220 Schottky | 3–5A SMD diode |
| Power filtering | Multi-stage, 3 barriers | Single bulk cap |
| Logic isolation | Dedicated 12V regulated rail | Direct from motor rail |
| MOSFET cooling | Active BLDC fan | None or passive heatsink |
| Component derating | 50–90% derating | 80–95% of rated limits |
| Gate drive | Series R + pull-down | Direct or oversized R |

Commercial modules cut corners on cost. This circuit prioritises **reliability through derating and isolation**.

---

## 📊 Filtering Strategy

Every capacitor targets a specific noise frequency range:

| Stage | Components | Target Frequency | Purpose |
|---|---|---|---|
| Input bulk | C1, C2 (1000µF each) | Low freq (50Hz–1kHz) | Smooths supply ripple |
| Input HF | C3 (1µF) | Mid frequency | Bridges bulk to HF range |
| Motor rail | L1 + D2 | High frequency | Attenuates PWM switching noise |
| Regulator output | C5, C4, C6 | Wide band | Post-regulation decoupling |
| 555 bulk | C7 (1000µF) | Low frequency | Stabilises 555 VCC |
| 555 HF | C8 (100nF) | High frequency | Suppresses fast transients at VCC |
| Timing | C9 (10nF) | — | Stable PWM frequency |
| Control pin | C10 (100nF) | High frequency | Prevents noise modulating duty cycle |

---

## ⚙️ Simulation

The schematic includes an oscilloscope (**XSC1**) to monitor:
- PWM waveform shape and stability
- Duty cycle at various R1 positions
- Rise/fall times of the switching signal

Simulated in **Multisim**.

---

## 🔌 Connections

| Terminal | Connection |
|---|---|
| `+` Input | 20V DC positive |
| `−` Input | Ground / negative |
| Motor `A` | Motor terminal 1 |
| Motor `B` | Motor terminal 2 (connects to drain) |
| R1 wiper | Speed control potentiometer |

---

## ⚠️ Notes & Limitations

- **D1 (1N1199C)** is rated ~12A and is the current bottleneck at stock configuration. Replace with a higher-rated device for currents above 10A continuous.
- The **IRFZ44N** is rated 49A but is thermally limited in practice. With active fan cooling, 20–25A continuous is achievable.
- Ensure the **40A flyback diode (D7)** voltage rating is at least 40V (2× the supply voltage rule).
- Keep gate wiring short to minimise parasitic inductance working against R3's damping effect.

---


# 🔌 AEGIS Pinout & Hardware Interfacing Specification

Hardware connection, pin mapping, and electrical specifications for the **AEGIS (Adaptive Electronic Glare Intelligence System)** implemented on the **STM32 ARM Cortex-M Platform** (STM32F411CEU6 BlackPill / STM32F103 BluePill).

---

## 📌 Complete STM32 Microcontroller Pinout Table

| Peripheral / Signal | Component Interface | STM32 GPIO | Peripheral Channel | Logic Level | Operating Mode | Function |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **Ambient Light Sensor** | TEMT6000 + Diffuser (Wide Angle) | `PA1` | `ADC1_IN1` | 0 – 3.3V | Analog Input (DMA Burst) | Ambient twilight & daylight reference |
| **Forward Glare Sensor** | TEMT6000 + Collimator (Narrow) | `PA2` | `ADC1_IN2` | 0 – 3.3V | Analog Input (DMA Burst) | Oncoming vehicular high-beam detector |
| **Low-Beam Headlamp** | Optoisolated IRF3205 N-MOSFET | `PA8` | `TIM1_CH1` | 3.3V (5V Tolerant) | Alternate Function PWM | Main forward road illumination |
| **High-Beam Headlamp** | Optoisolated IRF3205 N-MOSFET | `PA9` | `TIM1_CH2` | 3.3V (5V Tolerant) | Alternate Function PWM | Long-range illumination (glare dipped) |
| **Diagnostic OLED (SCL)**| SSD1306 128×64 Module | `PB6` | `I2C1_SCL` | 3.3V | Open-Drain, 4.7k Pull-up | Telematics & HMI clock bus |
| **Diagnostic OLED (SDA)**| SSD1306 128×64 Module | `PB7` | `I2C1_SDA` | 3.3V | Open-Drain, 4.7k Pull-up | Telematics & HMI data bus |
| **Manual Override Stalk**| Automotive Cabin Switch | `PA0` | External Interrupt | 3.3V | Input with Internal Pull-up | Driver manual high/low lockout |
| **Diagnostic Buzzer** | Active 5V Piezo Buzzer | `PB0` | General Purpose OUT | 3.3V / 5V | Push-Pull Output | Fault annunciation & acoustic warning |

---

## ⚡ Power Supply & Optical Decoupling

### 1. Dual-Rail Automotive Power Conditioning
- **Input:** Vehicle 12V DC battery rail (nominal 11.5V – 14.4V with ISO 7637 transient suppression).
- **Primary Step-Down:** LM2596 switching buck regulator converts 12V ➔ 5.0V DC for MOSFET gate drivers and active buzzer.
- **Secondary Linear Regulation:** Low-dropout AMS1117-3.3V provides clean, low-noise 3.3V logic rails for the STM32 and analog sensors.
- **Analog Isolation:** Dedicated ceramic decoupling capacitors (100 nF || 10 µF) placed adjacent to the sensor VDD and analog reference pins to eliminate DC-DC switching ripple.

### 2. Optical Sensor Collimation
- **Ambient Sensor (`PA1`):** Equipped with a translucent dome diffuser offering an omnidirectional 120° acceptance angle facing upward toward the sky.
- **Glare Sensor (`PA2`):** Housed inside a black cylindrical 3D-printed collimation tube (15 mm depth, 5 mm aperture) restricting acceptance to a narrow ±7.5° forward cone aligned directly with oncoming vehicle lanes.

---

## 🗺️ Hardware Architecture Schematic

```text
  +-------------------------------------------------------------+
  |                   OPTICAL FRONT-END                         |
  |                                                             |
  |   Ambient Sensor (Wide Angle) -------> [PA1] (ADC1_IN1)     |
  |   Glare Sensor (Collimated) ----------> [PA2] (ADC1_IN2)     |
  +-------------------------------------------------------------+
                               |
                               v
  +-------------------------------------------------------------+
  |                 STM32 ARM CORTEX-M CORE                     |
  |                                                             |
  |   - DMA Circular Buffer (8-sample burst)                    |
  |   - Q15 Fixed-Point 1st-Order IIR Filter (-26.4 dB @ 100Hz) |
  |   - Dynamic Ambient Floor Estimator                         |
  |   - Dual Asymmetric FSM (450ms dip / 2000ms restore)        |
  |   - Continuous Plausibility Diagnostics (20 Hz)             |
  +-------------------------------------------------------------+
           |                             |               |
           v                             v               v
  +-------------------+       +--------------------+  +---------+
  | POWER DRIVERS     |       | HMI DISPLAY        |  | OVERRIDE|
  | [PA8] Low Beam    |       | [PB6] I2C1_SCL     |  | [PA0]   |
  | [PA9] High Beam   |       | [PB7] I2C1_SDA     |  +---------+
  | (IRF3205 MOSFETs) |       | (SSD1306 128x64)   |
  +-------------------+       +--------------------+

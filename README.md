# CHILD / ADULT DETECTION SYSTEM

## 1. Abstract

This project presents an embedded system capable of distinguishing between a child and an adult entering a designated zone using two HC-SR04 ultrasonic sensors, an I2C LCD display, and an active buzzer — all interfaced with the STM32 Nucleo-F446RE microcontroller.

The system uses sensor height profiling to classify the detected individual and provides real-time visual feedback on the LCD along with an audible alert when a child is detected, making it suitable for safety-critical environments such as industrial zones, machinery areas, and restricted spaces.

---

## 2. Objectives

- Detect presence of a person entering a monitored area using ultrasonic ranging.
- Classify the detected person as a child or adult based on sensor height profiling.
- Display real-time detection status on a 16x2 I2C LCD.
- Trigger an active buzzer alert when a child is detected.
- Implement hardware-accurate microsecond timing using TIM1 for reliable sensor operation.
- Design a modular, maintainable codebase using STM32 HAL drivers.

---

## 3. System Overview

The system uses two HC-SR04 ultrasonic sensors placed at different heights:

- Lower sensor — approximately 60 cm (child height)
- Upper sensor — approximately 150 cm (adult chest height)

By analysing which sensors detect an object within range, the microcontroller determines whether the person is a child or adult.

### Hardware Prototype

The implemented hardware prototype consists of the STM32 Nucleo-F446RE microcontroller, ultrasonic sensor, I2C LCD, breadboard, jumper wires, and supporting components.

<img width="960" height="1280" alt="prathana project" src="https://github.com/user-attachments/assets/9a987886-44f2-4b3e-ac5c-0fd97b1de87e" />


---

## 3.1 Detection Logic

| Lower Sensor | Upper Sensor | Classification | Buzzer |
|---|---|---|---|
| Triggered (<40 cm) | No detection (>80 cm) | CHILD DETECTED | ON |
| Triggered (<40 cm) | Triggered (<80 cm) | ADULT DETECTED | OFF |
| No detection | No detection | NO PERSON | OFF |

---

## 4. Components

| Component | Qty | Purpose |
|---|---:|---|
| STM32 Nucleo-F446RE | 1 | Main microcontroller (Cortex-M4, 84 MHz) |
| HC-SR04 Ultrasonic Sensor | 2 | Distance measurement at two heights |
| I2C LCD 16x2 (PCF8574) | 1 | Real-time status display |
| Active Buzzer | 1 | Audible alert for child detection |
| 1 kΩ Resistor | 2 | Voltage divider for ECHO pin protection |
| 2 kΩ Resistor | 2 | Voltage divider (5V to 3.3V) |
| Breadboard + Jumper Wires | 1 set | Circuit prototyping |

---

## 5. Hardware Connections

### 5.1 STM32 Pin Assignment

| STM32 Pin | Connected To | Direction |
|---|---|---|
| PA1 | Lower sensor TRIG | Output |
| PA2 | Lower sensor ECHO (via 1kΩ + 2kΩ divider) | Input |
| PA3 | Upper sensor TRIG | Output |
| PA4 | Upper sensor ECHO (via 1kΩ + 2kΩ divider) | Input |
| PA5 | Buzzer + | Output |
| PB6 | LCD SCL | I2C Clock |
| PB7 | LCD SDA | I2C Data |
| 5V | Sensor VCC + LCD VCC | Power |
| GND | All component GNDs | Common Ground |

### 5.2 ECHO Voltage Divider

The HC-SR04 ECHO pin outputs 5V but the STM32 GPIO is only 3.3V tolerant.

A resistor voltage divider is used on both ECHO lines:

- 1 kΩ resistor in series
- 2 kΩ resistor to GND
- Produces approximately 3.33V at the STM32 pin

Connecting ECHO directly without this divider risks permanent damage to the microcontroller.

---

## 6. Software Design

### 6.1 Peripherals Configured

- **I2C1** — 100 kHz standard mode, PB6/PB7, for LCD communication
- **TIM1** — Internal clock, Prescaler = 83, 1 µs per tick at 84 MHz for ultrasonic timing
- **GPIO PA1, PA3, PA5** — Push-pull outputs for TRIG signals and buzzer
- **GPIO PA2, PA4** — Floating inputs for ECHO signals

### 6.2 Key Functions

| Function | File | Description |
|---|---|---|
| `lcd_init()` | `lcd_i2c.c` | Initialises LCD in 4-bit I2C mode |
| `lcd_send_string()` | `lcd_i2c.c` | Sends a character string to LCD |
| `lcd_put_cur()` | `lcd_i2c.c` | Sets cursor row and column position |
| `delay_us()` | `main.c` | Hardware-accurate microsecond delay via TIM1 |
| `ultrasonic_read()` | `main.c` | Triggers sensor, measures echo pulse, returns cm |

---

## 6.3 Main Loop Flow

1. Read lower sensor distance using `ultrasonic_read(PA1, PA2)`.
2. Read upper sensor distance using `ultrasonic_read(PA3, PA4)`.
3. Compare both values against thresholds:
   - Lower sensor: 40 cm
   - Upper sensor: 80 cm
4. Update LCD with detection result:
   - `CHILD DETECTED`
   - `ADULT DETECTED`
   - `NO PERSON`
5. Set or reset buzzer PA5 based on classification.
6. Repeat every 400 ms.

---

## 7. Expected Results

| Scenario | LCD Row 1 | LCD Row 2 | Buzzer |
|---|---|---|---|
| Child walks in (lower only) | CHILD DETECTED | !! DANGER !! | ON |
| Adult walks in (both sensors) | ADULT DETECTED | SAFE | OFF |
| No person in range | NO PERSON | Blank | OFF |
| System startup | SYSTEM READY | — | OFF |

---

## 8. Applications

- Industrial safety zones — alert when a child enters hazardous machinery areas
- School or daycare access control — monitor entry points
- Home automation — differentiate children from adults for smart device control
- Elevator safety systems — detect child presence for special operating modes
- Retail environments — age-based access restriction to certain sections

---

## 9. Conclusion

This project successfully demonstrates a low-cost, real-time child/adult detection system using the STM32 Nucleo-F446RE microcontroller and standard off-the-shelf components.

The use of dual ultrasonic sensors at different heights provides reliable height-based classification without requiring complex image processing or machine learning.

The system is modular, well-structured, and can be extended with additional features such as:

- Data logging
- Wireless alerts
- Camera integration
- More advanced deployments

---

## Project Information

**Platform:** STM32 Nucleo-F446RE  
**Language:** Embedded C (HAL)  
**IDE:** STM32CubeIDE  
**Project Type:** Embedded Systems  
**Year:** 2026

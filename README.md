# Industrial Crane Load & Collision Safety Guard

## 💡 Overview

This project implements a comprehensive safety and monitoring system for industrial crane hoisting installations. Using a **PIC18F4580 microcontroller**, the system handles real-time load weight monitoring via ADC and emergency crash preemption using hardware interrupts.

The firmware prevents catastrophic damage from structural cable overloading or carriage rail overruns by continuously scanning sensor inputs, updating an LCD dashboard, driving multi-state LED warnings, and actively overriding motor controls during critical faults.

## 🛠️ Hardware Requirements

* **Microcontroller:** PIC18F4580
* **Display:** 16x2 Character LCD
* **Sensors:** * Potentiometer (simulating the load cell/weight sensor)
* Limit Switch / Push Button (simulating the carriage bumper)


* **Actuators:** DC Motor with L293D driver (or Relay) for hoist mechanism
* **Indicators:** Green, Yellow, and Red LEDs; Piezo Buzzer
* **Controls:** Dedicated Reset Push Button

## 📌 Pin Configuration

| Component | Functionality | PIC18F4580 Pin | Notes |
| --- | --- | --- | --- |
| **Load Potentiometer** | Analog Load Input | `RA0 (AN0)` | Scaled to 0-100% via ADC |
| **Bumper Limit Switch** | Collision Interrupt | `RB0 (INT0)` | Triggers External Interrupt 0 on falling edge |
| **Reset Push Button** | System Reset | `RB1` | Digital GPIO with external pull-up resistor |
| **Hoist Motor Enable** | Motor Control | `RC0` | Commands the L293D driver or relay |
| **Green LED** | Safe Indicator | `RC1` | Active High |
| **Yellow LED** | Warning Indicator | `RC2` | Active High |
| **Red LED** | Overload Indicator | `RC3` | Active High |
| **Buzzer** | Audio Alert | `RC4` | Piezo element control |
| **LCD Data (D0-D7)** | 8-bit Parallel Bus | `PORTD (RD0-RD7)` | LCD Data lines |
| **LCD RS & EN** | Control Lines | `RE0`, `RE1` | Register Select and Enable strobe |

## ⚙️ System Logic & Operational States

The system operates in a continuous polling loop for weight management, but relies on absolute hardware interrupts (`INT0`) for collision preemption.

| System State | Trigger Condition | Output Actions | LCD Dashboard |
| --- | --- | --- | --- |
| **Normal Load** | ADC Load: 0% to 70% | • Green LED is ON.<br>

<br>• Motor is ENABLED. | **R1:** `LOAD: xx%`<br>

<br>**R2:** `STATUS: SAFE` |
| **Heavy Load Warning** | ADC Load: 71% to 90% | • Yellow LED flashes.<br>

<br>• Buzzer outputs a slow chirping signal. | **R1:** `LOAD: xx%`<br>

<br>**R2:** `STATUS: WARNING` |
| **Structural Overload** | ADC Load: > 90% | • Red LED is ON.<br>

<br>• Buzzer sounds continuously.<br>

<br>• Motor is DISABLED (forced low). | **R1:** `LOAD: xx%`<br>

<br>**R2:** `OVERLOAD - STOP!` |
| **Collision Stop** | Bumper switch triggers `INT0` | • Motor DISABLED instantly via ISR.<br>

<br>• Red & Yellow LEDs flash an alternating strobe (0xCC/0x33).<br>

<br>• System halts. | **R1:** `CRITICAL COLLISION`<br>

<br>**R2:** `SYSTEM HALTED!` |

### 🔄 Collision Reset Mechanism

Once the crane triggers the **Collision Stop** state, the system locks up for safety. The software continuously polls the dedicated Reset input switch (`RB1`). The crane remains totally unresponsive to load changes until the operator manually presses the Reset switch, which clears the alert flags and reinitializes the main monitoring loop.

## 🧮 ADC Scaling Logic

The PIC18F4580's ADC is configured to read an 8-bit result (0 to 255). To map this raw data to a user-friendly percentage for the LCD, the firmware utilizes the following mathematical scaling:

```c
// Convert 8-bit ADC resolution to a 0-100% scale
Percentage = (adc_value * 100) / 255;

```

## 🧑‍💻 Firmware Architecture

The code was written from scratch in **MPLAB X IDE** using the **XC8 Compiler**. Key implementation details include:

* **Configuration Bits:** Watchdog Timer disabled (`WDT = OFF`), Low-Voltage Programming disabled (`LVP = OFF`), and High-Speed Oscillator configured.
* **ADC Module:** `ADCON1` configured to set `RA0` as analog while keeping all other required output pins digital. `ADCON0` manages the channel selection and sampling.
* **Interrupt Service Routine (ISR):** Global (`GIE`) and Peripheral (`PEIE`) interrupts are enabled. The `__interrupt()` routine strictly listens for the `INT0IF` flag. Upon triggering, it immediately cuts power to the motor pins before handling the visual UI updates.
* **LCD Drivers:** Custom functions for `lcd_cmd()`, `lcd_data()`, and `lcd_init()` to handle the 8-bit initialization and data writing sequences.

## ▶️ Usage & Simulation

1. Open the project in **MPLAB X IDE**.
2. Compile the code using the XC8 toolchain to generate the `.hex` file.
3. Load the `.hex` file into your Proteus simulation or flash it directly to your PIC18F4580 development board.
4. **Testing:**
* Adjust the potentiometer to observe the state transitions from *Safe* -> *Warning* -> *Overload*.
* Trigger the bumper switch at any time to verify the hardware interrupt instantly overrides the active motor state.
* Press the Reset button to recover from a halted collision state.

## Circuit Diagram
![image alt](https://github.com/Gokulxu/PIC_PROJECTS/blob/25432d0c4f82eebd7129ab3573b24f573ef2de04/Screenshot%202026-06-16%20195905.png)

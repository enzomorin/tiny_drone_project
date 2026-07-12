# FPV Racing Drone - PCB components

Main MCU - STM32F405RGT6V
crystal oscillator 8MHz ; ESR ≤ 30–50 Ω
power management etc see diagram

Uni direc mosfet brushed motor driver

IMU ICM-42688-P

BMP388

AT7456E

RadioMaster RP1 2.4ghz ExpressLRS ELRS Nano récepteur pour TX16S ZORRO TX12 ELRS Version - radioo no firmeware needed only in it. - aliexpress
Antenne Lumenier AXII 2 5.8 GHz RHCP MMCX Droit


firmware main : betaflight

Top      → Signals + components
Inner 1  → GND (solid plane)
Inner 2  → Power (3.3V / 5V / VBAT)
Bottom   → Signals


FINAL LIST NEED TO CLEAN/ 
🧠 1. MAIN FLIGHT CONTROLLER (your PCB core)
MCU (brain)
STM32F405RGT6V
👉 Runs Betaflight
IMU (flight sensor — critical)
ICM-42688-P
👉 Gyro + accelerometer (SPI, high performance)
OSD (telemetry on video)
AT7456E
👉 Draws Betaflight data on FPV video
Blackbox (flight logging)
16 MB SPI Flash (W25Q128 or similar)
Barometer (optional but useful)
BMP388
GPS (optional)
u-blox M10
Power system
5V buck converter (FC + camera + receiver)
3.3V LDO (IMU + SPI devices)
1000 µF low ESR capacitor (VERY important)
Battery voltage divider → ADC pin
Current sensor (shunt or Hall)
Clock
8 MHz crystal (MCU)
optional 32 kHz crystal
🔌 2. RADIO CONTROL (868 MHz system — your custom link)

This is your most important communication system.

RF chip
Semtech SX1262
👉 868 MHz long-range RF
Receiver MCU (ON DRONE)

Best choice:

STM32G071 (recommended)
or
STM32G431 (faster, more headroom)
Receiver output to flight controller

Use:

UART → CRSF protocol
TX Controller → 868 MHz RF → Drone Receiver → CRSF UART → STM32F405

Betaflight reads it as:

Serial RX = CRSF
Antenna (868 MHz)

You must use an external antenna:

Best options:
8.6 cm wire whip (quarter-wave)
dipole antenna (better stability)

👉 DO NOT try PCB antenna first

🎮 3. TRANSMITTER (controller you HOLD)

Since you are building your drone first:

BEST option (buy, don’t build first):
RadioMaster Pocket ELRS

or better:

RadioMaster Boxer ELRS
Why this matters:
already supports CRSF
already supports ELRS ecosystem
saves you months of RF development
📷 4. FPV VIDEO SYSTEM (ANALOG — SIMPLE & PROVEN)
Camera
RunCam Phoenix 2
Video transmitter (VTX)
TBS Unify Pro32 Nano
FPV antenna
Lumenier AXII 2
Video chain
Camera → AT7456E (OSD) → VTX → 5.8 GHz antenna → Goggles
⚡ 5. MOTOR SYSTEM (brushed motors)

Since you use brushed motors:

For each motor:

N-channel logic MOSFET
gate resistor (~10–100 Ω)
pull-down resistor (~10 kΩ)
flyback protection (optional depending motor type)

Control:

PWM from STM32F405 timers
🔗 6. FULL SYSTEM ARCHITECTURE
                 ┌────────────────────────┐
                 │  RADIO CONTROL 868 MHz │
                 │ SX1262 + STM32G071    │
                 └──────────┬─────────────┘
                            │ CRSF UART
                            ▼
                    STM32F405 (Betaflight)
                 ┌────────────────────────┐
                 │ IMU: ICM-42688-P       │
                 │ OSD: AT7456E          │
                 │ Flash: Blackbox        │
                 │ GPS (optional)         │
                 └──────────┬─────────────┘
                            │ PWM
                            ▼
                     Brushed Motors (MOSFETs)

CAMERA PATH:
Camera → AT7456E → 5.8 GHz VTX → Goggles
🧠 7. WHAT YOU SHOULD BUILD FIRST (IMPORTANT)

Don’t build everything at once.

Step order:
1️⃣ Flight controller PCB
STM32F405
IMU
power
USB
basic UART
2️⃣ Add receiver (868 MHz)
SX1262 + STM32G071
CRSF output
3️⃣ Add FPV video
camera + AT7456E + VTX
4️⃣ Add motors last
🚀 Key truth (important)
868 MHz = CONTROL (long range)
5.8 GHz = VIDEO (standard FPV)
STM32F405 = flight brain
CRSF = communication protocol between receiver and Betaflight
If you want next step

I can help you design:

🔧 1. Full PCB schematic (Betaflight-ready F405 board)
🔧 2. 868 MHz receiver PCB (SX1262 layout)
🔧 3. MOSFET motor driver circuit (brushed motors)
🔧 4. Or full wiring + PCB placement guide (very important for drones)

Just tell me 👍

Yes — you’ve basically got the right idea, but let’s make it precise and correct so you don’t design something that’s hard to debug later.

🧠 1. Your two MCUs = two separate firmware projects

You have:

🟢 Flight controller

STM32F405RGT6V
→ runs Betaflight

🟡 Receiver module (868 MHz)

Semtech SX1262 + STM32G071/G431
→ runs ExpressLRS or custom firmware

🔌 2. How you flash them (THIS IS THE KEY POINT)
🟢 A) STM32F405 (Flight controller)
✔ You use:
USB (main method)
DFU bootloader (built into STM32)
Betaflight Configurator
Wiring needed:
USB D+
USB D−
5V input
GND
optional BOOT0 button
Flow:
PC → USB → STM32F405 → Betaflight firmware

👉 This is your main and easiest flashing method

🟡 B) Receiver MCU (STM32G071 / STM32G4)
✔ You use:
SWD (Serial Wire Debug)
Required pins:
SWDIO
SWCLK
GND
3.3V reference
RESET (optional but recommended)
Tool:
ST-Link V2 or V3 programmer
Flow:
PC → ST-Link → SWD → STM32G071 → Receiver firmware
⚠️ 3. Important correction (your assumption)

You said:

“one SWD and the other USB”

✔ This is mostly correct, BUT:

Device	USB	SWD
STM32F405 FC	✔ primary	✔ backup/debug
Receiver MCU	❌ usually no USB	✔ required

👉 So actually:

FC = USB primary
RX = SWD primary
🧠 4. Best professional setup (what FPV boards do)
Flight controller (F405)

You include BOTH:

USB-C (for flashing + config)
SWD pads (for recovery/debug)
Receiver board
NO USB
only SWD pads
🔧 5. Why SWD is essential (don’t skip this)

If you design your own PCB, SWD lets you:

recover a bricked MCU
debug firmware step-by-step
flash without bootloader issues
test SPI (SX1262 communication)

👉 Without SWD, custom hardware becomes painful.

📡 6. Where SX1262 fits

Semtech SX1262

Important:

SX1262 has NO firmware
it is NOT programmable like STM32
it only talks via SPI to your receiver MCU

So:

SX1262 ↔ SPI ↔ STM32G071 ↔ UART (CRSF) ↔ STM32F405

1. Analog video path (what you must have)

To get video working, you need this chain:

FPV Camera → Video Transmitter (VTX) → FPV Goggles/Receiver

✔ FPV Camera

You need an analog FPV camera (not digital like DJI / HDZero unless you change system).

Look for:

5V or VBAT powered FPV camera
NTSC/PAL output (most support both)
Example types: Caddx Ratel / RunCam Phoenix (analog versions)

Camera output gives:

Video signal (CVBS)
Ground
✔ Video Transmitter (VTX)

This is essential—you cannot skip it.

You need:

5.8GHz analog VTX
25mW–800mW (or more depending on legality)
SmartAudio or IRC Tramp control (optional but recommended)

Connect:

Camera video → VTX video in
Ground shared
Power (VBAT or 5V depending on model)
✔ Antenna

Don’t forget:

VTX antenna (SMA / RP-SMA / U.FL depending on VTX)

Without this you will get:

overheating VTX
almost no range
2. Your OSD chip (AT7456E)

You chose AT7456E, which is exactly what Betaflight uses for analog OSD.

Important:

The OSD is NOT in the camera or VTX — it sits inline.

Correct wiring:
Camera → AT7456E → VTX

So you need:

Camera video IN → AT7456E VIN
AT7456E VOUT → VTX video IN
3. Connection to STM32F405

Your MCU (STM32F405RGT6) communicates with OSD via SPI.

Typical pins:

SCK
MOSI
CS (chip select)
GND
5V or 3.3V (check board level shifting)

Betaflight already supports this OSD chip natively.

4. Power system (VERY important)

You need a clean power setup:

Required:
LiPo battery (3S–6S typical)
Voltage regulator (BEC):
5V for camera + MCU (if needed)
9V/12V optional for camera/VTX if supported
Strong recommendation:

Use separate filtered power for video

Why:

reduces “jello” noise / stripes in video
5. RC control system (you didn’t mention this)

Betaflight requires receiver input:

You need ONE of:

ELRS receiver (recommended)
Crossfire receiver
FrSky receiver (older)

Connect to STM32 via:

UART (CRSF / SBUS)
6. Sensors (for Betaflight flight control)

Minimum for stable flight:

Gyro + accelerometer (usually MPU6000 or BMI270 on FC)
Optional but useful:
Barometer (altitude hold)
Magnetometer (rare in FPV)
7. Other important missing parts

These are often forgotten:

✔ Capacitor on battery input
1000µF–2200µF low ESR capacitor
Reduces voltage spikes → protects VTX + MCU
✔ Current sensor (optional but useful)
For battery monitoring
✔ Beeper (optional)
For crash recovery
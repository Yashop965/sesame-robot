# Sesame Robot — Project Overview

> **Master prompt for AI models:** This file contains everything an AI needs to
> understand, build, modify, and deploy the Sesame Robot project. Feed this file
> to any code-generating AI as context before asking it to write or debug firmware.

---

## 1. Project Summary

**Sesame Robot** is an open-source mini quadruped robot — a 4-legged walker built
around an ESP32 microcontroller. It uses 8 servo motors (2 per leg) for movement
and has an OLED face for expressive animations.

- **Forked from:** [dorianborian/sesame-robot](https://github.com/dorianborian/sesame-robot)
- **Repo location:** `D:\sesame-robot\`
- **Build status:** In progress — firmware not yet written/adapted for our board

---

## 2. Hardware Bill of Materials

### Owned / Purchased (arriving tomorrow)

| Item | Qty | Notes |
|------|-----|-------|
| ESP32-D0WD DevKit V1 (DOIT variant) | 1 | On COM7, CH340 USB bridge, 4MB flash |
| MG90S all-metal micro servos (180°) | 8 | Leg actuators, metal gears, 4.8–6V |
| 0.96" SSD1306 I2C OLED (128x64) | 1 | Face display, I2C addr 0x3C |
| USB-C data/power cable | 1 | 5V/3A for tethered operation & flashing |
| 22AWG & 30AWG silicone wire | 1 kit | Power/signal wiring |
| Heat-shrink assortment | 1 | Insulate joints |
| Small zip ties | 1 pack | Wire management |
| Small protoboard (~5×7 cm) | 1 | For hand-wired distro board |
| 3-pin male headers | 8 | Servo breakout connectors |
| M2 × 5mm self-threading screws | ~40 | All plastic/rubber joints |
| M2.5 × 5mm machine screws | 10 | Servo horn attachment |
| Rocker power switch (KCD1) | 1 | Panel mount |
| Buck converter (5–12V → stable 5V/3A) | 1 | For future battery use |

### Not Yet Ordered

| Item | Reason |
|------|--------|
| 3D printed parts (11 STL files) | Budget constraint — will order ASAP |
| Battery (Bambu Lab 14500 7.4V 800mAh) | Future upgrade after tethered PoC |

**Note:** The robot will run **USB-C tethered** initially (no battery). The buck
converter and battery are for the wireless upgrade phase.

---

## 3. Board Details: ESP32 DevKit V1 (DOIT)

| Property | Value |
|----------|-------|
| Chip | ESP32-D0WD rev1.0 |
| Cores | Dual Core + LP Core @ 240MHz |
| Port | COM7 |
| USB Bridge | CH340 (QinHeng) |
| Flash | 4MB (3.3V QIO @ 80MHz) |
| FQBN | `esp32:esp32:esp32` |
| Upload Speed | 921600 baud |
| Operating Voltage | 3.3V logic, 5V USB input |

**IMPORTANT GPIO restrictions on ESP32:**
- GPIO 6, 7, 8, 9, 10, 11 are connected to the SPI flash — **DO NOT USE**
- GPIO 1, 3 are UART TX/RX — avoid if possible
- All other GPIOs are safe for servo control

---

## 4. Pinout — Distro Board V1 (Hand-Wired)

This is the wiring we will use since we have ESP32 DevKit V1 (not S2/S3).

### Servo Motor Connections (8 channels)

| Servo # | Name | GPIO | Function | Leg |
|---------|------|------|----------|-----|
| 1 | R1 | 15 | Hip rotation | Right-front |
| 2 | R2 | 2 | Knee flexion | Right-front |
| 3 | L1 | 23 | Hip rotation | Left-front |
| 4 | L2 | 19 | Knee flexion | Left-front |
| 5 | R4 | 4 | Hip rotation | Right-rear |
| 6 | R3 | 16 | Knee flexion | Right-rear |
| 7 | L3 | 17 | Hip rotation | Left-rear |
| 8 | L4 | 18 | Knee flexion | Left-rear |

### OLED Display (128x64 I2C)

| Signal | GPIO |
|--------|------|
| SDA | 21 |
| SCL | 22 |
| I2C Address | 0x3C |

### Power

- **Current:** USB-C 5V tethered (via CH340)
- **Target:** 5V 3A capable, buck converter for future battery
- Servos draw power directly from 5V rail (not from ESP32 3.3V)
- Common ground between ESP32 and servo power

---

## 5. Servo Orientation & Leg Mapping

Each leg has **2 servos**:
- **Hip servo** (top): Rotates the leg forward/backward
- **Knee servo** (bottom): Bends the leg

```
       Front
    R1(L)──L1(R)
   /            \
R2               L2
 |     BODY      |
L4               R4
   \            /
    L3(R)──R3(L)
       Rear
```

Neutral (rest position): All servos at 90°.
Stand position: ~135/45° alternating pattern on hips/ knees.
Angles coded in `movement-sequences.h`.

---

## 6. Software Stack

### Firmware (Arduino / C++)

The upstream firmware architecture:

| File | Purpose |
|------|---------|
| `sesame-firmware-main.ino` | Main entry: setup(), loop(), WiFi, API, OLED init, servo PWM |
| `movement-sequences.h` | All poses (stand, walk, dance, etc.) and gait cycles |
| `face-bitmaps.h` | 128×64 OLED face bitmap data (~30 face expressions) |
| `captive-portal.h` | Web UI HTML/CSS/JS for phone control |

### Required Arduino Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| ESP32Servo | **v3.0.9** (pinned!) | Servo PWM. **Do NOT use newer** — bug #103 affects multi-servo writes |
| Adafruit SSD1306 | latest | OLED display driver |
| Adafruit GFX Library | latest | Graphics primitives for OLED |
| WiFi (built-in) | — | Access Point + Station mode |
| WebServer (built-in) | — | HTTP API server |
| DNSServer (built-in) | — | Captive portal DNS |
| ESPmDNS (built-in) | — | mDNS service discovery (`sesame-robot.local`) |

### Key Firmware Changes Needed for DevKit V1

The upstream firmware is configured for **Lolin S2 Mini** by default. For our
ESP32 DevKit V1, we must change:

1. **servoPins** → use Distro Board V1 array: `{15, 2, 23, 19, 4, 16, 17, 18}`
2. **I2C pins** → use `I2C_SDA 21`, `I2C_SCL 22`
3. **Board settings** → ESP32 Dev Module, not S2 Mini
4. No USB CDC On Boot needed (only S2/S3 needs that)

### Motor Tester (Debugging)

A standalone `sesame-motor-tester.ino` exists for testing servos via serial:
- `id,angle` → e.g. `0,90` moves motor 0 to 90°
- `all,angle` → moves all motors
- `stop` → detaches (powers down) all motors

### Communication / API

The robot creates a WiFi Access Point named `Sesame-Controller` (pass: `12345678`)
and serves a web-based remote control. It also exposes a REST API:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/` | GET | Web UI (captive portal) |
| `/cmd?go=forward` | GET | Movement commands |
| `/cmd?pose=wave` | GET | Pose commands |
| `/cmd?motor=1&value=90` | GET | Direct servo control |
| `/api/status` | GET | JSON status |
| `/api/command` | POST | JSON command + face control |
| `/getSettings` | GET | Current settings |
| `/setSettings` | POST | Update settings |

---

## 7. Output Goals

### Phase 1 — Minimum Viable Robot (MVP)
- [ ] Motor tester firmware uploaded and verified (each servo responds)
- [ ] All 8 servos wired and moving correctly
- [ ] OLED display working (shows "Sesame" boot screen)
- [ ] Basic stand pose achieved (robot stands on its own)
- [ ] Walk forward/backward working (even if clumsy)
- [ ] Turn left/right functional

### Phase 2 — Full Upstream Feature Parity
- [ ] WiFi AP mode working
- [ ] Web controller accessible from phone
- [ ] All 17 poses working (wave, dance, bow, pushup, etc.)
- [ ] Animated OLED face expressions
- [ ] JSON API functional for external control

### Phase 3 — Polish & Extend
- [ ] Gait tuning for smooth walking
- [ ] Calibration routine (per-servo subtrim)
- [ ] Battery + buck converter integration
- [ ] 3D printed shell installed
- [ ] Optional: Python control scripts

---

## 8. Directory Structure

```
D:\sesame-robot\
├── about.md                  ← YOU ARE HERE
├── README.md                 ← Project readme
├── firmware/                 ← Arduino source code
│   ├── sesame-firmware-main.ino
│   ├── movement-sequences.h
│   ├── face-bitmaps.h
│   ├── captive-portal.h
│   └── debugging-firmware/
│       └── sesame-motor-tester.ino
├── hardware/                 ← Hardware reference
│   ├── pinout.md
│   ├── bom/README.md
│   ├── cad/                  ← STEP & Fusion 360 models
│   ├── pcb/                  ← Distro board Gerber files
│   └── printing/             ← 3D print guides & STLs
├── models/                   ← STL files for 3D printing (v117)
├── docs/                     ← Build guides (TBD)
└── .git/
```

---

## 9. Flash Instructions (for ESP32 DevKit V1)

```
Board:        ESP32 Dev Module
FQBN:         esp32:esp32:esp32
Port:         COM7
Upload Speed: 921600
Flash Mode:   QIO @ 80MHz
CPU Freq:     240MHz
```

Steps:
1. Open `sesame-firmware-main.ino` in Arduino IDE
2. Install libraries: ESP32Servo v3.0.9, Adafruit SSD1306, Adafruit GFX
3. Uncomment Distro Board V1 pinout lines
4. Select board: Tools → Board → ESP32 Arduino → ESP32 Dev Module
5. Select port: COM7
6. Upload
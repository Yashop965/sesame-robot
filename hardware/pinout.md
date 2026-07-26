# Sesame Robot — Hardware Reference

## Board: ESP32 DevKit V1 (DOIT)

| Property | Value |
|---|---|
| Chip | ESP32-D0WD rev1.0 |
| Cores | Dual Core + LP Core @ 240MHz |
| Port | COM7 |
| USB Bridge | CH340 (QinHeng) |
| MAC | 90:38:0c:b3:84:20 |
| Flash | 4MB (3.3V) |
| FQBN | `esp32:esp32:esp32` |
| Upload Speed | 921600 baud |
| Flash Mode | QIO @ 80MHz |

## Servo Pinout (Distro Board V1)

| Servo # | GPIO | Function |
|---------|------|----------|
| 1 | 15 | Leg 1 |
| 2 | 2 | Leg 1 |
| 3 | 23 | Leg 2 |
| 4 | 19 | Leg 2 |
| 5 | 4 | Leg 3 |
| 6 | 16 | Leg 3 |
| 7 | 17 | Leg 4 |
| 8 | 18 | Leg 4 |

## OLED (128x64 I2C)

| Signal | GPIO |
|--------|------|
| SDA | 21 |
| SCL | 22 |

## Power

- **Current setup:** USB-C tethered (no battery)
- **Target:** 5V 3A via USB

## Servos

- **Model:** MG90S (x8)
- **Type:** Micro servo, metal gears
- **Voltage:** 4.8–6V

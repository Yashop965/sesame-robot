# Sesame Robot — Execution Plan

**Goal:** Get the robot walking ASAP with minimum dependencies.
**Constraint:** 3D prints not ordered yet (funds), hardware arrives tomorrow.

---

## Phase 1 — Motor Tester & Hardware Verification (Day 1)

**Goal:** Confirm all electronics work before writing complex code.

### Step 1.1 — Assemble wiring
- Wire the protoboard with 3-pin headers for 8 servos
- Connect servo grounds together to ESP32 GND
- Connect servo power (red wires) to a separate 5V rail (NOT ESP32 3.3V)
- Connect servo signals to GPIOs 15, 2, 23, 19, 4, 16, 17, 18
- Wire OLED: SDA→21, SCL→22, VCC→3.3V, GND→GND

### Step 1.2 — Flash motor tester
- Open `firmware/debugging-firmware/sesame-motor-tester.ino`
- Uncomment Distro Board V1 pinout (`const int servoPins[8] = {15, 2, 23, 19, 4, 16, 17, 18};`)
- Upload to ESP32 at 921600 baud
- Open Serial Monitor at 115200 baud
- Run `0,90`, `1,90`, ... `7,90` to verify each servo moves

### Step 1.3 — Verify OLED
- Use an I2C scanner sketch or the motor tester (modify to init OLED)
- Confirm address 0x3C responds

**Deliverable:** All 8 servos and OLED confirmed working.

---

## Phase 2 — Stand Pose (Day 1–2)

**Goal:** First real firmware — robot stands up.

### Step 2.1 — Adapt main firmware for DevKit V1
- Open `sesame-firmware-main.ino`
- Uncomment Distro Board V1 pin array: `{15, 2, 23, 19, 4, 16, 17, 18}`
- Set `I2C_SDA 21`, `I2C_SCL 22`
- Set `AP_SSID` to `"Sesame-Robot"` (keep AP mode, no network for now)
- Upload and test

### Step 2.2 — Calibrate stand angles
- The stand pose in upstream code uses R1=135, R2=45, L1=45, L2=135, R4=0, R3=180, L3=0, L4=180
- These may need tuning for our specific servo orientation
- Use serial commands or web UI to tweak each servo angle

**Deliverable:** Robot stands stable on its own.

---

## Phase 3 — Basic Movement (Day 2–3)

**Goal:** Walking, turning — the robot moves.

### Step 3.1 — Walk forward
- Use the upstream `runWalkPose()` gait cycle
- Tune `frameDelay` and `walkCycles` for smooth walking
- Adjust per-leg angles if gait is off-balance

### Step 3.2 — Walk backward + turn
- Same gait logic, reversed direction
- Test turning left/right

### Step 3.3 — Add poses
- Enable: rest, stand, wave, dance
- Remaining poses can be added later

**Deliverable:** Robot walks forward/backward and turns on command.

---

## Phase 4 — Web Control & Polish (Day 3–4)

**Goal:** Full remote control from phone.

### Step 4.1 — WiFi AP + captive portal
- Test: connect phone to "Sesame-Robot" WiFi
- Open browser → captive portal should appear
- Test D-pad controls for movement
- Test pose buttons

### Step 4.2 — OLED face expressions
- Verify rest, stand, walk face bitmaps display correctly
- Add any missing bitmap data if needed

**Deliverable:** Phone-controlled robot with expressive face.

---

## Phase 5 — Assembly & 3D Printing (After Funds Available)

### Step 5.1 — Order prints
- Print all 11 STL files from `models/` or `hardware/printing/stl/`
- Recommended: PLA, 0.2mm layer height, supports for top cover

### Step 5.2 — Final assembly
- Mount servos into 3D printed frame
- Install OLED in top cover slot
- Route wiring through internal frame
- Install power switch
- Test all movements with the shell on

### Step 5.3 — Battery upgrade (optional)
- Add buck converter (5–12V → 5V/3A)
- Wire through rocker switch
- 7.4V Bambu Lab 14500 battery

---

## Phase 6 — Extras (Future)

- [ ] Gait optimization (smoother walking)
- [ ] Python control scripts via REST API
- [ ] Voice assistant integration
- [ ] Custom face designs
- [ ] Battery telemetry on OLED
- [ ] OTA firmware updates

---

## Quick Reference: Common Commands

### Flash motor tester
```
arduino-cli compile --fqbn esp32:esp32:esp32 firmware/debugging-firmware/
arduino-cli upload -p COM7 --fqbn esp32:esp32:esp32 firmware/debugging-firmware/
```

### Flash main firmware
```
arduino-cli compile --fqbn esp32:esp32:esp32 firmware/
arduino-cli upload -p COM7 --fqbn esp32:esp32:esp32 firmware/
```

### Serial monitor
```
python -m serial.tools.miniterm COM7 115200
```
(or use Arduino IDE Serial Monitor)

---

## Risk Register

| Risk | Mitigation |
|------|------------|
| Servo brownout (voltage drop) | Increase `motorCurrentDelay` in settings; use external 5V supply |
| Wrong servo orientation | Add servo `subtrim` array for per-servo offset |
| ESP32 can't drive 8 servos smoothly | Use hardware timers (already allocated); limit simultaneous moves |
| WiFi interference | AP mode is 2.4GHz only; keep distance from other APs |
| OLED I2C address mismatch | Run I2C scanner to verify (common alternatives: 0x3C or 0x3D) |
| 3D print quality issues | Print test cube first; use brim/supports per upstream guide |
# 🌀 OXiDrive — BLDC Smart Servo Actuator

> **Drop-in replacement for hobby servos. Real FOC. Real torque. Real robotics.**

**OXiDrive** is an open-source, STM32-based BLDC servo actuator with Field-Oriented Control (FOC) and magnetic encoder feedback. It is designed to fit standard hobby servo mounting patterns while delivering the performance and control modes demanded by modern robotics.

---

## Why OXiDrive?

Hobby servo motors (MG996R, DS3218, etc.) are universally available but fundamentally limited:

| Pain Point | Hobby Servo | OXiDrive |
|---|---|---|
| Control | Dumb PWM position only | Torque · Velocity · Position (FOC) |
| Feedback | None (open-loop) | 12–14-bit magnetic encoder |
| Smoothness | Jitter under load | Smooth, closed-loop FOC |
| Stall behaviour | Coil burn / gear strip | Current-limited stall protection |
| Torque range | 1–15 kg·cm | 10–30 kg·cm target |
| Advanced interface | PWM only | PWM + UART + CAN (roadmap) |
| Cost at performance | Cheap but limited | Accessible (made-in-India) |

Imported equivalents (Dynamixel, ODrive-based actuators) cost **₹50,000–₹1,00,000**. OXiDrive targets a fraction of that cost with an accessible, educationally transparent design.

---

## Target Specifications

| Parameter | Specification |
|---|---|
| Motor type | 3-phase BLDC (gimbal/outrunner class) |
| Torque output | 10–30 kg·cm (gearbox-dependent) |
| Operating voltage | 12–24 V |
| Controller | STM32G4 series (ARM Cortex-M4, FPU) |
| Gate driver | DRV8323 / FDMT80080 (3-phase) |
| Encoder | AS5600 / AS5048A (12–14-bit magnetic) |
| Gearbox | Optional planetary, 6:1 or 10:1 |
| Control modes | Position, Velocity, Torque (FOC) |
| Interfaces | PWM (servo-compat.), UART, CAN Bus (V2) |
| Footprint | MG996R-compatible mounting pattern |
| Firmware base | SimpleFOC (Arduino/PlatformIO) |
| Protection | Over-current, stall detection, soft-start/stop |

> **Note:** Specifications are prototype targets. Final values will be validated on hardware in Phase 4–5.

---

## Key Features

### Hardware
- 3-phase BLDC motor with optional planetary gearbox reduction (6:1 / 10:1)
- AS5600 / AS5048A magnetic encoder — 12–14-bit absolute position sensing, no index pulse required
- STM32G4-based controller — hardware floating-point, high-resolution timer for FOC PWM generation
- DRV8323 3-phase gate driver — integrated current sensing, SPI-configurable gain
- Standard MG996R bolt-hole pattern and connector pinout (5-pin: GND, 5V logic, PWM, TX, RX)

### Firmware
- Field-Oriented Control via SimpleFOC library (ported to STM32G4 / PlatformIO)
- Three operating modes selectable at boot or over UART:
  - **Position mode** — servo-style angle control via PWM or UART command
  - **Velocity mode** — RPM setpoint with torque clamping
  - **Torque mode** — direct current (torque) control for force-sensitive tasks
- Auto-calibration: encoder zero-offset and pole-pair detection on first boot
- Stall detection: current spike + zero-velocity watchdog → safe shutdown
- Soft-start and soft-stop ramps to prevent mechanical shock

### Interfaces
- **PWM** — standard 50 Hz, 1–2 ms pulse; maps to position mode (servo drop-in)
- **UART** — binary packet protocol at 115200 baud; full mode switching and telemetry
- **CAN Bus** *(V2 roadmap)* — multi-actuator bus for legged robots and arms

---

## Development Roadmap

```
Phase 1  ✅  Concept & Architecture
Phase 2  ✅  Documentation & Mockups
Phase 3  🟧  PCB Design — Rev 1
Phase 4  🟧  FOC Firmware Bring-up
Phase 5  🟧  Hardware Integration & Bench Testing
Phase 6  ☐   Closed-Loop Validation & Calibration Refinement
Phase 7  ☐   Campus Alpha Kit (5–10 units, field testing)
Phase 8  ☐   OXiDrive V2 — CAN Bus, refined PCB, production BOM
```

### Phase Details

| Phase | Status | Description |
|---|---|---|
| 1 — Concept | ✅ Done | Motor topology selection, controller choice (STM32G4), control mode architecture |
| 2 — Documentation | ✅ Done | README, block diagrams, component selection rationale, BOM draft |
| 3 — PCB Rev 1 | 🟧 Active | 4-layer PCB: power stage (DRV8323 + MOSFETs), STM32G4 MCU, encoder header, UART/CAN |
| 4 — Firmware | 🟧 Active | SimpleFOC port to STM32G4, closed-loop position/velocity/torque modes, UART protocol |
| 5 — Bench Testing | 🟧 Active | Motor spin-up, encoder calibration, back-EMF tuning, thermal characterisation |
| 6 — Validation | ☐ Planned | Load-cell torque measurement, step-response plots, stall/over-current trip testing |
| 7 — Campus Kits | ☐ Planned | 5–10 assembled units for external makers, feedback loop, curriculum material |
| 8 — V2 Release | ☐ Planned | CAN Bus node support, improved thermal pad, reduced BOM cost, miniaturised form-factor |

---

## Example Applications

- **Legged robots** — quadrupeds and hexapods requiring fast, smooth, torque-aware joints
- **Robotic arms** — 4–6 DOF manipulators with gravity compensation via torque mode
- **Animatronics** — smooth and silent actuation for expressive joints
- **Gimbal mechanisms** — velocity-mode control for stabilised camera platforms
- **Research platforms** — force-controlled experiments requiring calibrated torque output
- **Educational kits** — teach BLDC physics, FOC algorithms, embedded motor control

---

## Repository Structure *(planned)*

```
oxidrive/
├── hardware/
│   ├── pcb/            # KiCad schematic + PCB layout (Rev 1)
│   ├── mechanical/     # STEP/STL files, MG996R mounting adapter
│   └── bom/            # Component BOM (LCSC, JLCPCB)
├── firmware/
│   ├── src/            # SimpleFOC-based STM32G4 firmware
│   ├── platformio.ini
│   └── config/         # Motor & encoder calibration config headers
├── docs/
│   ├── wiring.md       # Pinout, power wiring diagram
│   ├── uart-protocol.md
│   └── calibration.md
└── README.md
```

---

## Getting Started *(after Phase 4)*

1. **Flash firmware** using ST-Link V2 and PlatformIO
2. **Connect encoder** (AS5600 via I²C on PA8/PA9)
3. **Power up** at 12–24 V through the JST-PH power connector
4. **Calibrate** — send `CAL\n` over UART; motor will spin one pole-pair cycle and lock encoder zero
5. **Control** via PWM (servo mode, no changes to your existing code) or UART commands

---

## Educational Mission

OXiDrive is designed to be fully transparent — every design decision is documented so students can learn:

- BLDC motor physics (back-EMF, Kv, pole pairs)
- Space-vector PWM and Field-Oriented Control (FOC) mathematics
- Absolute magnetic encoder integration (I²C / SPI, hall effect)
- 3-phase gate driver design (bootstrap circuitry, dead-time)
- Embedded motor control firmware (control loops, interrupts, timers)
- Robotics motion planning (position, velocity, torque modes)

---

## License

- **Firmware:** MIT License
- **Hardware (PCB, Schematics):** CERN Open Hardware Licence v2 — Permissive (CERN-OHL-P)

---

## Vision

> *OXiDrive aims to be India's first open smart-actuator platform —*
> *empowering students, makers, and robotics teams to build world-class robots at an accessible cost.*

---

**Current Status:** Phases 3–5 in parallel (PCB layout + firmware bringup + bench testing)

*Contributions, feedback, and campus partnerships welcome.*

# 🌀 OXiDrive — Modular Smart Actuator Platform

> **Round. Chainable. Wireless-ready. FOC-controlled. Built for real robotics.**

**OXiDrive** is an open-source modular smart actuator platform built around a **5010 BLDC motor** with a **cycloidal / harmonic / planetary gearbox**, housed in a **black anodized aluminium round enclosure**, controlled by an onboard **STM32G4 FOC controller** with absolute magnetic encoder feedback.

Actuators daisy-chain via circular **CAN Bus + Power connectors** for clean, scalable wiring. A dedicated **ESP32-based Wireless Bridge Module** sits at the head of the chain to deliver wireless command translation from the main robot controller — no wireless hardware required inside each actuator node.

---

## Concept & Vision

![OXiDrive Single Actuator Concept](robot%20actuator%20concept.jpg)
*OXiDrive actuator node — black anodized round housing, integrated gearbox, onboard FOC controller, chain connectors*

![OXiDrive Modular Chain Architecture](modular%20actuator%20concept.jpg)
*Modular distributed actuator network — ESP32 wireless bridge gateway → CAN chain → N identical actuator nodes*

---

## Why OXiDrive?

Modern high-performance actuators (Dynamixel Pro, Hebi Robotics, Moteus) cost **₹50,000–₹2,00,000** when imported and are entirely unavailable to most Indian students and makers. Hobby servos are cheap but fundamentally broken for real robotics:

| Pain Point | Hobby Servo | OXiDrive |
|---|---|---|
| Form factor | Rectangular, bulky | Compact round actuator (black anodized) |
| Control | PWM position only (open-loop) | Torque · Velocity · Position (FOC, closed-loop) |
| Feedback | None | 14-bit absolute magnetic encoder (output shaft) |
| Torque quality | Jitter, dead-zones, gear-strip on stall | Smooth, stiff, current-controlled torque |
| Communication | Single PWM wire | CAN Bus daisy-chain + UART + optional wireless |
| Multi-actuator wiring | Rats-nest individual wiring | Single looped chain: power + CAN in one pass |
| Wireless control | Not possible | ESP32 Wireless Bridge module at chain head |
| Cost at performance | Cheap but limited | Accessible — made-in-India, open hardware |

---

## System Architecture

OXiDrive is a **two-part system**:

```
┌─────────────────────┐
│  Main Robot         │
│  Controller / PC    │
└─────────┬───────────┘
          │  Wi-Fi / BLE (wireless)
          ▼
┌─────────────────────┐
│  OXiDrive Wireless  │
│  Bridge (ESP32)     │  ◄── USB-C (flash / host debug)
└─────────┬───────────┘
          │  CAN Bus + Power (wired chain)
          ▼
┌──────────┐    ┌──────────┐    ┌──────────┐             ┌──────────┐
│  Node 01 │───►│  Node 02 │───►│  Node 03 │───► ... ───►│  Node N  │
└──────────┘    └──────────┘    └──────────┘             └──────────┘
   ↑ CAN + Power daisy-chain via circular connectors (both sides)
```

### Part 1 — OXiDrive Actuator Node

Each actuator is a fully self-contained intelligent module:

- **5010 BLDC outrunner motor** — high torque density at low speed, designed for gearbox pairing
- **Integrated gearbox** — cycloidal (zero backlash), harmonic (high reduction, compact), or planetary (cost-effective modular) selected at build time
- **Black anodized aluminium round housing** — precision-machined, industrial finish, gearbox bolt-circle flange on output side
- **STM32G4 onboard FOC controller** — Cortex-M4 with FPU, high-resolution timers for >40 kHz PWM switching
- **AS5048A 14-bit absolute magnetic encoder** — mounted on the output shaft for true position sensing with no cumulative error
- **DRV8323 3-phase gate driver** — integrated current sensing, SPI-configurable gain, hardware fault protection
- **Chain connectors (both sides)** — single circular connector per side carries both CAN differential pair and power lines; enables a clean single-loop wiring pass through all joints

**Connectors exposed per actuator:**

| Connector | Type | Purpose |
|---|---|---|
| CAN In | Circular (chain) | CAN Bus + Power from upstream actuator or bridge |
| CAN Out | Circular (chain) | CAN Bus + Power to downstream actuator |
| UART | 3-pin header | Serial debug / direct single-unit control |
| SWD Pads | PCB pads (5) | SWDIO, SWDCK, RESET, GND, 3.3V — ST-Link flashing and live debug (developer only) |

No other connectors. No PWM. No external antenna. Clean.

---

### Part 2 — OXiDrive Wireless Bridge Module

A standalone custom ESP32-based PCB that acts as the **CAN master and wireless gateway** at the head of the actuator chain.

- **ESP32 chip options:**
  - **ESP32-C6** *(recommended)* — Wi-Fi 6 + BLE 5 + Thread/Zigbee; lowest latency for real-time robot control
  - **ESP32-S3** — Wi-Fi + BLE 5, dual-core; better for data-heavy telemetry or vision pipelines
  - **ESP32-C3** — Wi-Fi + BLE 5, single-core low-cost; suitable for simple setups or development
- **CAN transceiver** (SN65HVD230 or MCP2562) — bridges ESP32 TWAI peripheral to the actuator chain
- **Power regulation** — accepts 12–24 V from chain power input; regulates to 3.3 V for ESP32
- **USB-C** — host/flash port; also supports direct UART to CAN bridge via PC
- **Custom PCB** — compact industrial layout; not a generic dev board

The bridge connects to the first actuator in the chain over a wired CAN + Power cable. All downstream actuators communicate purely over CAN — **no wireless silicon inside any actuator node**.

---

## Target Specifications

### Actuator Node

| Parameter | Specification |
|---|---|
| Motor | 5010 BLDC outrunner (3-phase) |
| Gearbox | Cycloidal / harmonic / planetary (build variant) |
| Gear ratio (target) | 6:1 – 36:1 depending on gearbox selection |
| Torque output | 10–40 kg·cm (gearbox and ratio dependent) |
| Operating voltage | 12–24 V |
| Controller MCU | STM32G4 (Cortex-M4, FPU, HR timers) |
| Gate driver | DRV8323RS (3-phase, integrated 3× current sensing) |
| Encoder | AS5048A, 14-bit absolute, SPI |
| Control modes | Position · Velocity · Torque (FOC, closed-loop) |
| Communication | CAN 2.0B (daisy-chain) + UART |
| CAN node ID | Configurable via UART command or hardware ID pads |
| Firmware | SimpleFOC on STM32G4 (PlatformIO) |
| Housing | Round, black anodized aluminium |
| Termination | CAN termination resistor jumper on last node |
| Protection | Over-current, stall detection, thermal shutdown, soft-start/stop |

### Wireless Bridge Module

| Parameter | Specification |
|---|---|
| MCU | ESP32-C6 / S3 / C3 (variant-dependent) |
| Wireless | Wi-Fi 6 + BLE 5 (C6) / Wi-Fi + BLE 5 (S3, C3) |
| CAN interface | TWAI peripheral + SN65HVD230 or MCP2562 transceiver |
| Power input | 12–24 V from chain connector or 5V via USB-C |
| Power output | 3.3V regulated for ESP32; passes 12–24V to actuator chain |
| Bus role | CAN master / gateway node |
| Host interface | Wi-Fi, BLE, or USB-C UART |
| Connectors | USB-C (flash/host) + CAN Out + Power Out to chain |

> **Note:** All specifications are prototype targets. Values will be validated on hardware in Phases 5–6.

---

## Key Features

### Actuator Hardware
- **5010 BLDC** — large outrunner class; proven in drone/robotics applications; excellent torque-to-weight ratio when geared
- **Gearbox options (build variants):**
  - *Cycloidal:* Zero backlash, high shock resistance, compact; best for legged robots
  - *Harmonic:* Highest reduction ratios (50:1–100:1) in smallest envelope; best for precision arms
  - *Planetary:* Cost-effective, modular, well-understood; best for educational builds and arms
- **AS5048A encoder on output shaft** — eliminates gearbox backlash error from position measurement; true output-side closed-loop control
- **DRV8323RS gate driver** — hardware current sensing on all three phases; 8 V gate drive for fast MOSFET switching; SPI fault register
- **STM32G4** — 170 MHz Cortex-M4 with FPU; HRTIM peripheral for sub-nanosecond FOC PWM dead-time control
- **Circular chain connectors** — both sides of the housing; a single cable loop carries 12–24 V power **and** CAN differential pair through every joint in the chain

### Firmware
- Field-Oriented Control via SimpleFOC (ported to STM32G4 / PlatformIO)
- Three operating modes — selectable via UART command or CAN packet:
  - **Position mode** — closed-loop angle control with configurable PID + feedforward
  - **Velocity mode** — RPM setpoint with torque clamping
  - **Torque mode** — direct current/torque control for compliant or force-sensitive joints
- Auto-calibration on first boot: pole-pair detection, encoder zero offset, encoder direction
- Stall detection: current spike + zero-velocity watchdog → safe controlled shutdown
- Soft-start and soft-stop ramps to prevent mechanical shock on joints
- CAN node ID configurable via UART; stored in onboard flash

### Connectivity
- **CAN Bus (primary)** — multi-drop daisy-chain; each node is CAN-addressed; supports synchronised multi-axis setpoint commands
- **UART** — single-unit debug, configuration, calibration, and direct control
- **SWD pads** — 5 pads (SWDIO, SWDCK, RESET, GND, 3.3V) for ST-Link firmware flashing and GDB live debug; developer access only; not a user-facing connector
- **Wireless** — via dedicated Wireless Bridge Module only; zero wireless hardware inside the actuator node

---

## Development Roadmap

```
Phase 1   ✅  Concept & Architecture
Phase 2   ✅  Hardware Design Decisions & Documentation
Phase 3   🟧  Actuator PCB Rev 1 (STM32G4 + DRV8323 + AS5048A + CAN)
Phase 4   🟧  FOC Firmware Bring-up (spin, calibrate, close all loops)
Phase 5   🟧  Mechanical Integration (5010 + gearbox + housing + connectors)
Phase 6   ☐   Closed-Loop Validation & Performance Characterisation
Phase 7   ☐   Wireless Bridge Module (ESP32 PCB + CAN gateway firmware)
Phase 8   ☐   Full System Integration (multi-node chain + wireless end-to-end)
Phase 9   ☐   Campus Alpha Kit (5–10 assembled systems + 1 bridge per kit)
Phase 10  ☐   OXiDrive V2 (CANFD, miniaturised PCB, final gearbox, production BOM)
```

### Phase Details

| Phase | Status | Deliverable |
|---|---|---|
| 1 — Concept | ✅ Done | Motor (5010 BLDC), controller (STM32G4), form factor (round chain), gearbox selection rationale |
| 2 — Documentation | ✅ Done | README, architecture diagrams, component rationale, BOM draft |
| 3 — Actuator PCB | 🟧 Active | 4-layer KiCad PCB: DRV8323 power stage, STM32G4, AS5048A SPI, UART, CAN transceiver, SWD pads |
| 4 — FOC Firmware | 🟧 Active | SimpleFOC on STM32G4: motor spin, encoder calibration, position/velocity/torque loops, CAN node |
| 5 — Mechanical | 🟧 Active | 5010 motor mount, gearbox integration, round housing STEP files, circular connector mounting |
| 6 — Validation | ☐ Planned | Load-cell torque measurements, step-response characterisation, stall/over-current trip, thermal profiling |
| 7 — Wireless Bridge | ☐ Planned | ESP32 custom PCB, Wi-Fi/BLE host interface, CAN gateway firmware, protocol translation testing |
| 8 — System Integration | ☐ Planned | End-to-end chain test: wireless host → bridge → 3+ nodes via CAN; synchronised multi-axis control |
| 9 — Campus Alpha | ☐ Planned | 5–10 assembled actuator systems + 1 wireless bridge per kit; external maker feedback loop |
| 10 — V2 Release | ☐ Planned | CANFD support, miniaturised PCB, final gearbox selection, reduced BOM cost, public open hardware release |

---

## Example Applications

- **Legged robots** — quadrupeds and hexapods with torque-controlled, back-drivable joints; CAN chain maps cleanly to limb topology
- **Robotic arms** — 4–7 DOF manipulators with gravity compensation and force sensing via torque mode
- **Humanoids** — high-torque joints in a compact round form factor chainable across the entire structure
- **Animatronics** — smooth, silent, precise joint actuation with wireless operation via bridge
- **Research platforms** — force-controlled experiments requiring calibrated, transparent torque output
- **Educational kits** — hands-on learning for BLDC physics, FOC, CAN Bus, and embedded motor control

---

## Repository Structure *(planned)*

```
oxidrive/
├── hardware/
│   ├── actuator-pcb/       # KiCad schematic + PCB for actuator node (Rev 1)
│   ├── wireless-bridge/    # KiCad schematic + PCB for ESP32 bridge module
│   ├── mechanical/         # STEP/STL: round housing, gearbox interface, connector mount
│   └── bom/                # Component BOM (LCSC / JLCPCB compatible)
├── firmware/
│   ├── actuator/           # STM32G4 FOC firmware (SimpleFOC / PlatformIO)
│   │   ├── src/
│   │   ├── platformio.ini
│   │   └── config/         # Motor params, encoder config, CAN node ID
│   └── wireless-bridge/    # ESP32 Wi-Fi/BLE to CAN gateway firmware
│       ├── src/
│       └── platformio.ini
├── docs/
│   ├── architecture.md     # System architecture, block diagrams
│   ├── can-protocol.md     # CAN packet format, node IDs, command set
│   ├── uart-protocol.md    # UART command set for direct control/debug
│   ├── wiring.md           # Power and CAN chain wiring guide
│   └── calibration.md      # Encoder and FOC calibration procedure
└── README.md
```

---

## Getting Started *(after Phase 4–5)*

### Single Actuator — Wired UART
1. Flash firmware via **ST-Link V2** on the SWD pads using PlatformIO
2. Connect **UART** (TX, RX, GND) to your PC or microcontroller at 115200 baud
3. Power the actuator at **12–24 V** via the chain power connector
4. Send `CAL\n` over UART — motor spins one pole-pair cycle and locks encoder zero
5. Send mode and setpoint commands over UART to control position, velocity, or torque

### Multi-Actuator Chain — Wired CAN
1. Flash each actuator with a unique **CAN node ID**: send `NODEID x\n` over UART
2. Daisy-chain actuators: **CAN Out → CAN In**, **Power Out → Power In** via the circular chain connectors
3. Place the **CAN termination jumper** on the last actuator in the chain
4. Send CAN packets from your host (or the wireless bridge) to address individual or all nodes

### Wireless Setup — Bridge Module
1. Flash the **ESP32 wireless bridge** via USB-C
2. Configure Wi-Fi SSID/password via USB serial on first boot
3. Connect **bridge CAN Out + Power Out** to the first actuator's CAN In + Power In
4. Connect your host PC or robot controller over **Wi-Fi / BLE** and send commands through the bridge

---

## Educational Mission

OXiDrive is fully transparent by design — every architectural decision is documented so students can learn:

- **BLDC motor physics** — Kv, back-EMF, pole pairs, three-phase commutation
- **Field-Oriented Control (FOC)** — Clarke/Park transforms, space-vector PWM, PID loop tuning
- **Absolute magnetic encoder** — AS5048A via SPI, interpolation, offset calibration, output-shaft sensing
- **3-phase gate driver design** — DRV8323, bootstrap circuitry, dead-time control, current sensing topology
- **CAN Bus fundamentals** — arbitration, node addressing, packet structure, daisy-chain topology
- **Wireless protocol bridging** — ESP32 TWAI peripheral, Wi-Fi/BLE to CAN translation, gateway design
- **Gearbox mechanical design** — cycloidal vs. harmonic vs. planetary trade-offs in real applications

---

## License

- **Firmware:** MIT License
- **Hardware (PCB, Schematics, Mechanical files):** CERN Open Hardware Licence v2 — Permissive (CERN-OHL-P)

---

## Vision

> *OXiDrive aims to be India's first open modular smart-actuator platform —*
> *empowering students, makers, and robotics teams to build world-class robots at an accessible cost.*

---

**Current Status:** Phases 3–5 running in parallel — actuator PCB layout · FOC firmware bringup · mechanical housing integration

*Contributions, feedback, and campus partnerships welcome.*

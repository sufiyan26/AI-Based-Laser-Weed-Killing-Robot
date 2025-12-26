# AI-Based Laser Weed-Killing Robot

This project implements an **AI-assisted agricultural robot** that distinguishes weeds from crops using an onboard vision module and executes precision laser weeding. The system combines a HuskyLens AI camera, Arduino-based control firmware, a tracked/wheeled mobile base, and a gated 1 W laser subsystem with multiple hardware and software safety interlocks.

## Core Concept & Objective

- **Goal:** Automate spot weeding in crop rows by targeting only weeds, reducing manual labor and avoiding chemical herbicides.
- **Key Idea:** Use HuskyLens for local object/feature recognition, feed detections to an embedded controller that validates targets, aligns the robot, and fires the laser only within a safe window and geometry.

## Technical Design

### Vision & Detection

- **Sensor:** HuskyLens AI vision module configured for object/color recognition of weed classes.
- Outputs bounding box coordinates, IDs, and confidence directly over UART/I²C.
- Avoids external compute by running all inference on the HuskyLens itself.

### Control Firmware (Arduino)

- **MCU:** Arduino Uno acting as the main controller.
- Receives detection data from HuskyLens (position, class, color).
- Implements a finite‑state machine:
  - `SEARCH` → `ALIGN` → `ARM` → `FIRE` → `SAFE`
- Performs:
  - Target validation (class = weed, confidence threshold).
  - Deadbanding and hysteresis to avoid jitter on borderline detections.
  - Timing windows and distance checks before enabling the laser.

### Mobility & Actuation

- **Drive:** L298N motor driver controlling DC motors (tracked or differential wheeled base).
- PWM‑based speed and direction control for smooth turns and row following.
- Alignment logic uses horizontal offset of weed in the HuskyLens frame to steer toward the target before firing.

### Laser Subsystem & Safety

- **Laser:** ~1 W laser module with dedicated driver circuit.
- Driver includes:
  - Enable gating controlled by MCU.
  - Current limiting and protective components.
- **Safety Interlocks:**
  - Laser can fire only when:
    - Valid weed detection is locked.
    - Robot motion is within a small velocity tolerance (nearly stationary).
    - Target lies inside an allowed “fire window” region in the image.
  - Software timeouts limit firing duration and duty cycle to avoid overheating.
  - Manual kill switch and emergency stop line.
  - No‑fire zones configured around crop centroid regions to prevent crop damage.

## Project Structure
├── firmware/
│ ├── main.ino # FSM, HuskyLens parsing, drive + laser control
│ └── huskylens_if.cpp # UART/I²C interface and detection parsing
├── hardware/
│ ├── laser_driver.sch # Laser driver + interlock circuit schematic
│ └── power_layout.pdf # 12 V distribution + regulators (logic vs motors)
└── docs/
├── fsm_design.md # State machine description and timing
└── safety_tests.md # Field safety and validation procedures



## Example Behavior

1. Robot moves along a crop row in **SEARCH** state.
2. HuskyLens detects a weed → controller enters **ALIGN**, steering until weed is centered.
3. When aligned and within range, robot transitions to **ARM** and then briefly to **FIRE**, pulsing the laser on the weed.
4. System returns to **SAFE** and continues forward, skipping crops and ignoring low‑confidence detections.
5. Manual kill switch or any safety violation instantly disables the laser and stops motion.

## Key Results

- Demonstrated reliable discrimination between trained weed targets and crops in test plots.
- Achieved targeted weed ablation with minimal collateral exposure thanks to strict fire windows and duty‑cycle limits.
- Validated full loop from AI detection → navigation alignment → safety checks → controlled laser firing.

## Skills Demonstrated

Embedded C on Arduino • AI vision integration (HuskyLens) • Finite‑state machine design • Motor control with L298N (PWM, direction) • Power distribution and grounding for mixed analog/digital loads • High‑power laser driver design with interlocks • Safety‑critical embedded system design • Field testing and validation


# ADAS Collision Avoidance – CAPL Simulation

**ADAS Collision Avoidance System Simulation** implemented using **CANoe CAPL** with a **multi-ECU architecture**.
This project demonstrates sensor fusion, time-to-collision (TTC) assessment, AEB (Automatic Emergency Braking), evasive steering control, and fault-injection–based fail-safe handling.

---

## Project Overview

This repository contains a **CANoe-based ADAS simulation environment** that models a real-world collision avoidance system.

### Key characteristics

* Multi-ECU architecture (ADAS, Radar, Camera, Chassis, EPS, ESC, HMI, Diagnosis)
* Sensor fusion (Radar + Camera)
* TTC-based risk assessment
* AEB braking and evasive steering control
* Fault injection (sensor fault, message drop/delay, timeout)
* Fail-safe state machine with diagnostic logging

> The goal of this project is to demonstrate **ADAS system design and validation capability**, not just CAPL syntax usage.

---

## System Architecture

### ECUs & Responsibilities

| ECU                  | Description                                                                      |
| -------------------- | -------------------------------------------------------------------------------- |
| **ADAS_ECU**         | Main logic, state machine, sensor fusion, TTC calculation, AEB & evasive control |
| **SimulatorNode**    | Driver input simulation (steering, brake, override)                              |
| **SensorECU_Radar**  | Radar sensor simulation with drop/delay fault injection                          |
| **SensorECU_Camera** | Camera sensor simulation (lane availability, object detection)                   |
| **ChassisECU**       | Vehicle dynamics simulation (speed, gear, ignition)                              |
| **EPS_ECU**          | Electric Power Steering simulation                                               |
| **ESC_ECU**          | AEB braking & wheel-speed simulation                                             |
| **HMI_ECU**          | Warning display & logging                                                        |
| **DiagnosisECU**     | DTC accumulation and fault duration tracking                                     |

---

## Data Flow

### CAN Message Flow

```
Radar / Camera / Vehicle / Driver
        │
        ▼
     ADAS_ECU
        │
        ├─ Steering Command → EPS_ECU
        ├─ Brake Command    → ESC_ECU / ChassisECU
        └─ Status & Warning → HMI / Diagnosis
```

### System Variable (sysvar) Flow

* **Panel → CAPL** : Driver inputs, sensor configuration, fault injection
* **CAPL → Panel** : ADAS state, TTC, warnings, steering angle, wheel speeds

---

## ADAS State Machine

| State        | Description                                      |
| ------------ | ------------------------------------------------ |
| **Standby**  | System inactive                                  |
| **Armed**    | Monitoring environment                           |
| **Risk**     | Collision risk detected (TTC threshold exceeded) |
| **Evasive**  | AEB braking and/or evasive steering active       |
| **FailSafe** | Sensor/communication fault detected              |

### State Transitions (Simplified)

* `Standby → Armed` : IGN ON + Gear D + Vehicle moving
* `Armed → Risk` : Collision risk detected
* `Risk → Evasive` : TTC ≤ threshold & conditions satisfied
* `Any → FailSafe` : Sensor fault or communication timeout

---

## Collision Risk Logic

### TTC (Time-To-Collision)

```
TTC = Fused_Distance / Relative_Speed
```

### Risk Levels

| TTC     | Action                          |
| ------- | ------------------------------- |
| > 2.5 s | No risk                         |
| ≤ 2.5 s | Forward Collision Warning (FCW) |
| ≤ 1.5 s | High risk                       |
| ≤ 0.8 s | AEB / Evasive control           |

---

## AEB & Evasive Control

### AEB Braking Levels

| Level   | Deceleration |
| ------- | ------------ |
| Level 1 | 2.0 m/s²     |
| Level 2 | 5.0 m/s²     |
| Level 3 | 8.0 m/s²     |

### Evasive Steering

* Activated only if:

  * Sensor fusion is valid
  * Lane availability detected
  * Driver override inactive
  * EPS ready and fault-free

---

## Fault Injection & Fail-Safe

Supported fault scenarios:

* Radar / Camera sensor fault
* Message drop (random loss)
* Message delay
* Communication timeout (watchdog)
* Sensor disagreement

Fail-safe behavior:

* Immediate state transition to **FailSafe**
* Reduced braking applied
* Diagnostic Trouble Codes (DTC) logged

---

## Repository Structure

```
adas-collision-avoidance-capl/
├─ dbc/
│  └─ ADAS_CollisionAvoidance.dbc
├─ sysvar/
│  └─ ADAS_CollisionAvoidance.sysvar
├─ capl/
│  ├─ ADAS_ECU.can
│  ├─ SimulatorNode.can
│  ├─ SensorECU_Radar.can
│  ├─ SensorECU_Camera.can
│  ├─ ChassisECU.can
│  ├─ EPS_ECU.can
│  ├─ ESC_ECU.can
│  ├─ HMI_ECU.can
│  └─ DiagnosisECU.can
├─ docs/
│  ├─ architecture.md
│  ├─ state-machine.md
│  ├─ fault-injection.md
│  └─ demo-scenarios.md
└─ README.md
```

---

## Tools & Technologies

* **Vector CANoe**
* **CAPL**
* **CAN / DBC**
* **ADAS System Design**
* **State Machine Modeling**
* **Fault Injection & Validation**

---

## What This Project Demonstrates

* Realistic ADAS system modeling
* CAPL-based multi-ECU coordination
* Safety-oriented state machine design
* Validation-focused fault handling
* Clear separation of responsibilities across ECUs

---

## Demo Scenarios (Recommended)

* Normal driving → Risk → FCW → AEB
* Radar fault → Camera-only fusion
* Dual sensor fault → FailSafe
* Driver override during evasive maneuver

---

## Author

This project was created as a **portfolio demonstration** of
**ADAS system design, CAPL implementation, and validation logic**.

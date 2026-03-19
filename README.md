# 🏭 Multitask Pneumatic and Conveyor System

> **A fully automated multi-station industrial pick-and-place system** combining PLC logic, pneumatic actuators, and stepper-driven conveyors for sequential object handling and sorting.

---

## 📸 Demo & Media

> 📷 *Add your system images here*
> 🎥 *Add your demo video link here (YouTube / Google Drive)*
---

## 📌 Project Overview

This project implements a **3-station automated conveyor and pneumatic handling system** controlled by a **Siemens S7-200 PLC**. Objects travel across three conveyors, getting picked, placed, sorted, and pushed by various pneumatic mechanisms at each station — all sequenced through ladder logic and a detailed state-machine flowchart.

| Parameter | Details |
|---|---|
| **Controller** | Siemens S7-200 PLC |
| **Programming Language** | Ladder Diagram (LAD) |
| **Conveyor Drive** | 3× NEMA 17 Stepper Motors + TB6600 Drivers |
| **Pneumatic Power** | 25L Air Compressor |
| **Power Supply** | SMPS 12V & 24V DC |
| **Valve Type** | 3/5 Way Solenoid Valves |
| **States / Steps** | 43 (M0.0 → M4.2) |

---

## 🔧 System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                    MULTITASK CONVEYOR SYSTEM                         │
│                                                                      │
│ [CONVEYOR 01]──────────────►[CONVEYOR 02]──────────────►[CONVEYOR 03]│
│ Stepper+TB6600              Stepper+TB6600             Stepper+TB6600│
│ S1 · S2 · S3                S4 · S5 · S6                    S7       │
│                   │                │               │                 │
│             Pick & Place    Pneumatic Arm       Push Unit            │
│             (TN01+Rodless+  (CY01+CY02+         (TN02)               │
│             Suction)         Gripper)                                │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🚦 Station Descriptions

### Station 1 — Conveyor 01 + Pick & Place Unit

- **Conveyor 01** is driven by a **NEMA 17 stepper motor** via a **TB6600 driver**.
- Three proximity sensors detect objects at the **start (S1)**, **middle (S2)**, and **end (S3)** positions.
- When an object reaches **S3 (end)**, the pick-and-place mechanism activates:
  - **TN01** (double-acting cylinder with linear guide) extends downward.
  - **Rodless cylinder** traverses horizontally to position the head.
  - **Suction cup** picks up the object.
  - The mechanism then transfers the object to **Conveyor 02**.

### Station 2 — Conveyor 02 + Pneumatic Arm

- **Conveyor 02** is driven by a second **NEMA 17 stepper motor** via a **TB6600 driver**.
- Three proximity sensors at **start (S4)**, **middle (S5)**, and **end (S6)**.
- At the **middle (S5)** position, the **pneumatic arm** operates:
  - **CY02** (double-acting cylinder with linear guide) moves vertically.
  - **CY01** (double-acting cylinder with linear guide) moves horizontally.
  - **Gripper** clamps and releases the object for repositioning / sorting.
- At the **end (S6)** of Conveyor 02, **TN02** (double-acting cylinder) pushes the object onto Conveyor 03.

### Station 3 — Conveyor 03 (Output)

- **Conveyor 03** is driven by a third **NEMA 17 stepper motor** via a **TB6600 driver**.
- One proximity sensor at the **end (S7)** detects object arrival.
- When **S7** triggers, the full cycle completes and the system returns to idle.

---

## 🔩 Hardware Components

### Mechanical & Pneumatic

| Component | Type | Qty | Notes |
|---|---|---|---|
| Conveyor Belts | Custom frame | 3 | Driven by NEMA 17 |
| NEMA 17 Stepper Motor | Bipolar | 3 | One per conveyor |
| TB6600 Stepper Driver | Microstepping | 3 | One per motor |
| TN01 Cylinder | Double-Acting | 1 | Pick & place vertical axis |
| TN02 Cylinder | Double-Acting | 1 | Push unit at end of CNV02 |
| Rodless Cylinder | Linear actuator | 1 | Horizontal traverse for pick unit |
| CY01 Cylinder | Double-Acting + Linear Guide | 1 | Pneumatic arm horizontal |
| CY02 Cylinder | Double-Acting + Linear Guide | 1 | Pneumatic arm vertical |
| Suction Cup | Vacuum gripper | 1 | Object pickup on pick & place |
| Gripper | Pneumatic finger | 1 | Object handling on arm |
| Air Compressor | 25L tank | 1 | Pneumatic power source |
| Solenoid Valves | 3/5 Way | Multiple | One per actuator direction |

### Sensors

| Sensor | Address | Location |
|---|---|---|
| S1 | I0.0 | CNV01 Start |
| S2 | I0.1 | CNV01 Middle |
| S3 | I0.2 | CNV01 End |
| S4 | I0.3 | CNV02 Start |
| S5 | I0.4 | CNV02 Middle |
| S6 | I0.5 | CNV02 End |
| S7 | I0.6 | CNV03 End |
| C1S1 (CY01 S1) | I1.2 | CY01 Backward limit |
| C1S2 (CY01 S2) | I1.1 | CY01 Forward limit |
| C2S1 (CY02 S1) | I1.0 | CY02 Up limit |
| C2S2 (CY02 S2) | I1.3 | CY02 Down limit |

### Outputs / Actuators

| Symbol | Address | Function |
|---|---|---|
| CNV_1 | Q1.0 | Conveyor 01 Motor |
| CNV_2 | Q1.2 | Conveyor 02 Motor |
| CNV_3 | Q1.4 | Conveyor 03 Motor |
| RODLESS | Q0.0 | Rodless Cylinder |
| TN_1 | Q0.1 | TN01 Cylinder |
| SUCTION | Q0.2 | Suction Cup Valve |
| CY_1 | Q0.3 | CY01 Cylinder |
| CY_2 | Q0.4 | CY02 Cylinder |
| GRIPPER | Q0.5 | Gripper Valve |
| TN_2 | Q0.6 | TN02 Push Cylinder |

### Electrical

| Component | Specification |
|---|---|
| PLC | Siemens S7-200 |
| Power Supply | SMPS 12V DC (logic) + 24V DC (outputs) |
| Solenoid Control | 24V DC via PLC outputs |
| Stepper Control | TB6600 with step/direction signals from PLC |

---

## 💻 PLC Program Structure

The ladder diagram is implemented in **Siemens STEP 7 Micro/WIN** for the S7-200 PLC (OB1 / MAIN block). It uses a **sequential state machine** driven by memory bits (M registers).

### State Machine Flow

```
M0.0 (INIT) → M0.1...M0.7 (CNV01 + Pick & Place)
             → M1.0...M1.7 (CNV02 + Pneumatic Arm descent)
             → M2.0...M2.7 (Arm pick cycle)
             → M3.0...M3.7 (Arm place + CNV02 push)
             → M4.0...M4.2 (CNV03 output)
             → Back to M0.0
```

### Key Networks Summary

| Network | State | Action |
|---|---|---|
| 1–2 | M0.0 | Initialize — all outputs OFF, set M0.1 |
| 3 | M0.1 | Wait for S1 (object on CNV01 start) |
| 4 | M0.2 | Start CNV01 (2s timer) |
| 5–6 | M0.3–M0.4 | Wait S2, pulse CNV01 (5s OFF → ON) |
| 7–8 | M0.5–M0.6 | Wait S3, stop CNV01 |
| 9 | M0.7 | Rodless ON → TN01 ON → Suction ON → TN01 OFF → Rodless OFF → TN01 ON → Suction OFF → TN01 OFF |
| 10–11 | M1.0–M1.1 | Wait S4, run CNV02 2s |
| 12–13 | M1.2–M1.3 | Wait S5 (NOT), stop CNV02 |
| 14–17 | M1.4–M1.7 | Check arm home, lower CY02, close gripper, raise CY02 |
| 18–23 | M2.0–M2.5 | Extend CY01, lower CY02, open gripper, retract |
| 24–27 | M2.6–M3.1 | Re-pick cycle with gripper |
| 28–33 | M3.2–M3.7 | Return arm, release, start CNV02, push with TN02 |
| 34–35 | M4.0–M4.1 | Wait S6, start CNV03 + TN02 |
| 36 | M4.2 | Wait S7 → cycle complete → return to M0.0 |

---

## 📂 Repository Structure

```
multitask-pneumatic-conveyor/
├── README.md
├── docs/
│   ├── flowchart.pdf          # Full state machine flowchart
│   ├── ladder_diagram.pdf     # Complete ladder diagram (22 pages)
├── plc/
│   ├── project instruction list.awl   # Exported PLC program (STEP 7 AWL/LAD)
│   ├── Project ladder.mwp          # Exported PLC program (STEP 7 AWL/LAD)
├── media/
│   ├── images/
│   │   ├── system_overview.jpg
│   └── demo_video.mp4         # (or link in README)
└── hardware/
    └── io_mapping.csv         # I/O address table
```

---

## ▶️ Getting Started

### Prerequisites

- Siemens STEP 7 Micro/WIN software
- Siemens S7-200 PLC with programming cable (PC/PPI)
- 25L air compressor (min. 6 bar)
- SMPS 24V DC power supply
- 3× TB6600 stepper drivers configured for NEMA 17

### PLC Upload Steps

1. Open plc/project instruction list.awl and Project ladder.mwp in **STEP 7 Micro/WIN**.
2. Verify I/O symbol table matches the address mapping above.
3. Connect to the S7-200 via PC/PPI cable.
4. Download the program block to the PLC.
5. Switch PLC to **RUN** mode.

### Startup Sequence

1. Power on SMPS (24V and 12V rails).
2. Start the air compressor — wait for tank pressure to reach operating level.
3. Power on the PLC — system initializes at **M0.0** (all outputs OFF).
4. Place an object at the start of Conveyor 01.
5. The system runs the full automated cycle.

---

## 📊 System Flowchart

> See [`docs/flowchart.pdf`](docs/flowchart.pdf) for the complete state-machine flowchart covering all 43 steps (M0.0 → M4.2).

---

## 📋 Ladder Diagram

> See [`docs/ladder_diagram.pdf`](docs/ladder_diagram.pdf) for the full 22-network S7-200 ladder program.

---

## 🙋 Author

**KEERTHI P.E.D.S.**
Assignment 02 — Multitask Pneumatic and Conveyor System

---

## 📄 License

This project is submitted as an academic assignment. All rights reserved by the author.

---

> *Feel free to open an issue or reach out if you have questions about the ladder logic or hardware setup.*

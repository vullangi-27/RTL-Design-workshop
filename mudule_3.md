# SKY130 CMOS Inverter — SPICE Simulation, Layout & Parasitic Extraction

> Module 3 VLSI Physical Design Laboratory using the SkyWater **SKY130** open-source PDK.

---

## Table of Contents
- [Aim](#aim)
- [Project Overview](#project-overview)
- [Objectives](#objectives)
- [Part 1 — SKY130_03_SK1: Labs for CMOS Inverter](#part-1--sky130_03_sk1-labs-for-cmos-inverter)
  - [1.1 Pre-Lab Concepts](#11-pre-lab-concepts)
  - [1.2 SPICE Deck Creation](#12-spice-deck-creation)
  - [1.3 SPICE Simulation](#13-spice-simulation)
  - [1.4 Switching Threshold (Vm)](#14-switching-threshold-vm)
  - [1.5 Static & Dynamic Simulation](#15-static--dynamic-simulation)
- [Part 2 — SKY130_03_SK2: Inception of Layout](#part-2--sky130_03_sk2-inception-of-layout)
  - [2.1 Active Regions](#21-active-regions)
  - [2.2 N-Well and P-Well Formation](#22-n-well-and-p-well-formation)
  - [2.3 Gate Terminal Formation](#23-gate-terminal-formation)
  - [2.4 Lightly Doped Drain (LDD)](#24-lightly-doped-drain-ldd)
  - [2.5 Source–Drain Formation](#25-sourcedrain-formation)
  - [2.6 Local Interconnect](#26-local-interconnect)
  - [2.7 Higher-Level Metal Routing](#27-higher-level-metal-routing)
  - [2.8 SKY130 PDK Basics](#28-sky130-pdk-basics)
  - [2.9 Standard-Cell Layout](#29-standard-cell-layout)
- [Part 3 — SKY130_03_SK3: Technology File Labs](#part-3--sky130_03_sk3-technology-file-labs)
  - [3.1 Final SPICE Netlist](#31-final-spice-netlist)
  - [3.2 Inverter Characterization](#32-inverter-characterization)
  - [3.3 Introduction to Magic](#33-introduction-to-magic)
  - [3.4 SKY130 PDK Introduction](#34-sky130-pdk-introduction)
  - [3.5 Magic → SPICE Extraction Flow](#35-magic--spice-extraction-flow)
  - [3.6 Fixing the poly.9 DRC Error](#36-fixing-the-poly9-drc-error)
  - [3.7 Synthesis, Floorplan & GRIDBOX Statistics](#37-synthesis-floorplan--gridbox-statistics)
- [Design Flow Summary](#design-flow-summary)
- [Results](#results)
- [Conclusion](#conclusion)
- [Repository Structure](#repository-structure)
- [Tools Used](#tools-used)
- [References](#references)

---

## Aim
To design and analyze a CMOS inverter using SkyWater **SKY130** technology, perform SPICE simulation and switching analysis, create and verify the physical layout using **Magic**, and extract the circuit information back from the layout for cross-verification.

## Project Overview
This repository documents the complete **Module 3 SKY130 VLSI laboratory flow**, split into three linked sub-projects:

| Sub-project | Focus |
|---|---|
| `SKY130_03_SK1` | CMOS inverter theory + SPICE simulation labs |
| `SKY130_03_SK2` | Inception and construction of the physical layout |
| `SKY130_03_SK3` | SKY130 technology-file labs, Magic verification & extraction |

The work begins with CMOS inverter fundamentals and SPICE-level simulation, then moves into layout construction, Magic-based layout verification, parasitic/netlist extraction, and correction of layout-rule (DRC) errors.

## Objectives
- Understand the operation of a CMOS inverter.
- Create a transistor-level SPICE netlist.
- Run the circuit using **ngspice**.
- Obtain input/output voltage transfer characteristics.
- Determine the switching threshold voltage, **Vm**.
- Study static and dynamic (transient) behavior.
- Understand SKY130 CMOS layout layers.
- Build and inspect a layout using **Magic**.
- Extract a SPICE netlist directly from the layout.
- Identify and fix layout-rule (DRC) errors.
- Maintain project commands, scripts, and results on GitHub.

---

## Part 1 — SKY130_03_SK1: Labs for CMOS Inverter

### 1.1 Pre-Lab Concepts
Core concepts revised before the lab:

- PMOS and NMOS operation
- CMOS inverter topology
- VDD and GND rails
- Input and output nodes
- Pull-up network (PMOS)
- Pull-down network (NMOS)
- Logic `0` and Logic `1` levels
- Voltage Transfer Characteristic (VTC)

**CMOS Inverter structure**
- A CMOS inverter consists of one PMOS and one NMOS transistor.
- PMOS forms the **pull-up** path; NMOS forms the **pull-down** path.
- The input drives both transistor gates.
- The output is taken from the common drain node.
- PMOS source is tied to `VDD`; NMOS source is tied to `GND`.

### 1.2 SPICE Deck Creation
A SPICE deck is a plain-text file describing the circuit: devices, supply, input stimulus, and simulation commands.

```bash
vim sky130inv.spice
# edit the deck, then:
# Esc
# :wq
# Enter
```

<p align="center">
  <img width="800" alt="newlayout" src="https://github.com/user-attachments/assets/e2c0dd78-4dbb-4aa3-ab12-0e8afeabe7f5" /><br/>
  <img width="800" alt="sky130spice" src="https://github.com/user-attachments/assets/2aacfcea-eebe-4405-bff0-8b9efdc45534" />
</p>

**Figure 1.** Creation of the SKY130 CMOS inverter SPICE deck.

### 1.3 SPICE Simulation
Once the deck is created, simulate it with `ngspice`.

```bash
ngspice sky130inv.spice
# inside ngspice:
run
plot v(in) v(out)
quit
```

Expected inverter behavior:
- Low input → High output
- High input → Low output
- A transition region centered around the switching threshold

<p align="center">
  <img width="800" alt="sky130A_tech" src="https://github.com/user-attachments/assets/c2de5c00-f210-4bf1-bc79-87458e55ca43" />
</p>

**Figure 2.** SKY130 CMOS inverter SPICE simulation result.

### 1.4 Switching Threshold (Vm)
**Vm** is the input voltage at which the inverter transitions between logic states, identified near the point where `Vin ≈ Vout` on the VTC curve. The exact value depends on transistor sizing, supply voltage, and process corner.

| Parameter | Value |
|---|---|
| `VDD` | `__ V` *(fill from your run)* |
| `Vm`  | `__ V` *(fill from your run)* |

> ⚠️ Record the value obtained from your own simulation — do not enter an estimated or guessed `Vm`.

### 1.5 Static & Dynamic Simulation

**Static simulation** — studies DC behavior:
- Voltage Transfer Characteristic (VTC)
- Switching threshold
- Logic-high / logic-low regions

**Dynamic simulation** — studies the time-domain response to a changing input:
- Propagation delay
- Rise time
- Fall time
- Output waveform shape

```spice
.tran 0.1n 100n
```

```bash
ngspice sky130inv.spice
```

---

## Part 2 — SKY130_03_SK2: Inception of Layout

### 2.1 Active Regions
The active region is the semiconductor area where transistor source/drain regions form.
- NMOS has its own active region.
- PMOS has its own active region.

<p align="center">
  <img width="800" alt="sky130tech" src="https://github.com/user-attachments/assets/e1138343-f301-4ee7-8cfd-73e4611f257f" /><br/>
  <img width="800" alt="ngspice2" src="https://github.com/user-attachments/assets/0552de96-47d1-4ef2-be90-4323c7c7dc7f" />
</p>

### 2.2 N-Well and P-Well Formation
- **N-Well** — hosts PMOS devices.
- **P-Well** — hosts NMOS devices (process-dependent).

Wells provide the correct body/substrate environment for each transistor type.

### 2.3 Gate Terminal Formation
The gate forms where polysilicon crosses the active region:

```
Polysilicon + Active region crossing → MOS transistor gate
```

The gate controls conduction between the source and drain.

<p align="center">
  <img width="800" alt="Layout node/connection selection" src="https://github.com/user-attachments/assets/7e509893-e0ea-4169-b56d-e1df54117d63" />
</p>

**Figure 4.** Layout node/connection selection during CMOS inverter layout formation.

### 2.4 Lightly Doped Drain (LDD)
LDD is a lightly doped region near the source/drain edge of a MOS transistor.

**Purpose**
- Reduces the peak electric field near the drain.
- Improves long-term device reliability.

### 2.5 Source–Drain Formation
Source and drain regions are formed via implantation/doping:

| Terminal | Connection |
|---|---|
| PMOS source | `VDD` |
| NMOS source | `GND` |
| PMOS & NMOS drains | Output node |

<p align="center">
  <img width="800" alt="consoleforsky" src="https://github.com/user-attachments/assets/836e7c24-6352-4b4f-8163-d6c4bcc5cf7a" />
</p>

**Figure 6.** NMOS GND connection in the CMOS inverter layout.

### 2.6 Local Interconnect
Local interconnect links nearby device terminals and layout regions, reducing reliance on longer metal routing for short connections.

### 2.7 Higher-Level Metal Routing
Higher metal layers are primarily used for:
- Power distribution
- Signal routing
- Connections between distant regions

```
Lower layers   → local / device-level connections
Higher layers  → longer signal and power connections
```

> Use the actual SKY130 layer names as defined in your installed PDK/lab environment.

### 2.8 SKY130 PDK Basics
The SKY130 PDK provides:
- Device models
- Layer definitions
- Design rules
- Technology information
- Standard-cell-related data

This enables the design to target a real semiconductor manufacturing process.

<p align="center">
  <img width="700" alt="slowlib" src="https://github.com/user-attachments/assets/f6a4482e-e761-48e9-a2e4-36c3ce7f6885" />
  <img width="700" alt="typicallib" src="https://github.com/user-attachments/assets/89967293-0a9a-4055-9027-c747b2f83835" /><br/>
  <img width="700" alt="fastlib" src="https://github.com/user-attachments/assets/36a81246-f896-47a1-9880-6a163bae2e42" />
  <img width="700" alt="pshortlib" src="https://github.com/user-attachments/assets/cde833e3-4a11-4293-a7d0-53460b8cc7e6" />
</p>

### 2.9 Standard-Cell Layout
A standard cell is a reusable layout block with fixed dimensions and defined power connections. A CMOS inverter standard cell typically includes:
- PMOS and NMOS devices
- `VDD` rail
- `GND` rail
- Input connection
- Output connection
- Contacts
- Required metal layers

---

## Part 3 — SKY130_03_SK3: Technology File Labs

### 3.1 Final SPICE Netlist
The final SPICE netlist represents the circuit once the required device and layout information is finalized.

```bash
ngspice sky130inv.spice
```

<p align="center">
  <img width="800" alt="invspice" src="https://github.com/user-attachments/assets/fe4751c5-4f22-4125-81c1-df5010285a52" />
</p>

**Figure.** Modified/final SKY130 CMOS inverter SPICE deck.

### 3.2 Inverter Characterization
Characterization determines the key electrical parameters:
- Switching threshold
- Propagation delay
- Rise time
- Fall time
- Power-related behavior

> Only values obtained from actual simulation runs should be recorded — no estimated figures.

### 3.3 Introduction to Magic
**Magic** is a VLSI layout tool used for:
- Viewing layout layers
- Editing layout geometry
- Design Rule Checking (DRC)
- Inspecting node connections
- Extracting circuit information from the layout

```bash
magic -T <technology-file> <layout-file>
```

> Replace `<technology-file>` with the exact tech-file path available in your SKY130 PDK installation.

### 3.4 SKY130 PDK Introduction
The SKY130 PDK supplies the technology information the tools rely on:
- Technology/layer information
- Device models
- Design rules
- Physical-design information

### 3.5 Magic → SPICE Extraction Flow

```
Layout
   ↓
DRC / Inspection
   ↓
Extraction
   ↓
SPICE Netlist
   ↓
SPICE Simulation
```

Magic inspects the drawn layout and extracts a circuit-level representation for cross-checking against the original SPICE deck — completing the SPICE-extraction-from-layout loop for the SKY130 CMOS inverter.

### 3.6 Fixing the `poly.9` DRC Error
A DRC error indicates a violated layout design rule.

**Procedure**
1. Open the layout in Magic.
2. Run DRC.
3. Locate the highlighted error region.
4. Read the exact rule-violation message.
5. Modify the geometry per the SKY130 design rule.
6. Re-run DRC.
7. Repeat until all required errors are cleared.

> An error should be **fixed**, not hidden by simply deleting the offending layer.

### 3.7 Synthesis, Floorplan & GRIDBOX Statistics

<p align="center">
  <img width="800" alt="gridboxes" src="https://github.com/user-attachments/assets/17bfe2ce-a03b-4102-b249-d32aa7df8f58" />
</p>

**Synthesis**
<p align="center">
  <img width="800" alt="runsysnthesis" src="https://github.com/user-attachments/assets/7676540d-8b7a-4d7d-a6a1-b8d8c1a948f3" />
</p>

**Floorplan**
<p align="center">
  <img width="800" alt="floorplan" src="https://github.com/user-attachments/assets/683a604f-64e7-45a2-be2c-cdb05518f60e" />
</p>

**Statistics**
<p align="center">
  <img width="800" alt="sta" src="https://github.com/user-attachments/assets/95cd6e28-52cc-46c8-9954-f3d4e05b55cd" />
</p>

---

## Design Flow Summary

```
CMOS Inverter
     ↓
SPICE Netlist
     ↓
ngspice Simulation
     ↓
Vm / VTC / Timing
     ↓
SKY130 Layout
     ↓
Magic DRC
     ↓
SPICE Extraction
     ↓
Layout Verification
```

## Results

| Metric | Value |
|---|---|
| Supply voltage (`VDD`) | *fill in from your simulation* |
| Switching threshold (`Vm`) | *fill in from your simulation* |
| Propagation delay | *fill in from your simulation* |
| Rise time | *fill in from your simulation* |
| Fall time | *fill in from your simulation* |
| DRC status (post fix) | *fill in — e.g. clean / 0 errors* |

> All figures above should be populated only from actual lab outputs, not estimated values.

## Conclusion
This SKY130 project provides practical, hands-on understanding of the CMOS inverter design flow — spanning SPICE simulation, switching-threshold analysis, static and dynamic characterization, SKY130 layout formation, Magic-based layout inspection, DRC error correction (including the `poly.9` fix), and SPICE extraction from the finished layout. The screenshots captured throughout this repository serve as evidence of the simulation, layout, and extraction activities performed.

## Repository Structure
```
.
├── SKY130_03_SK1/        # SPICE deck, ngspice scripts, VTC results
├── SKY130_03_SK2/        # Layout inception files (Magic .mag, screenshots)
├── SKY130_03_SK3/        # Tech-file labs, DRC fixes, extracted netlists
├── images/                # Simulation/layout screenshots
└── README.md
```

## Tools Used
- **SkyWater SKY130** open-source PDK
- **ngspice** — SPICE circuit simulation
- **Magic VLSI** — layout editing, DRC, and extraction
- **Vim** — SPICE deck editing

## References
- SkyWater SKY130 Open PDK documentation
- ngspice user manual
- Magic VLSI layout tool documentation

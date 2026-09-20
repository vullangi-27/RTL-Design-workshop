<h1 align="center">SKY130 – Pre-Layout Timing Analysis</h1>
<p align="center"><b>Timing Libraries · Synthesis · OpenSTA · Clock Tree Synthesis</b></p>

<p align="center">
  <img alt="PDK" src="https://img.shields.io/badge/PDK-SkyWater%20SKY130-blue?style=flat-square">
  <img alt="Synthesis" src="https://img.shields.io/badge/Synthesis-Yosys-orange?style=flat-square">
  <img alt="Timing" src="https://img.shields.io/badge/STA-OpenSTA-brightgreen?style=flat-square">
  <img alt="Flow" src="https://img.shields.io/badge/Flow-OpenLane-informational?style=flat-square">
 
</p>

<p align="center"><i>A study of how timing information is prepared and analyzed before physical layout, using the SkyWater SKY130 PDK, OpenSTA, and OpenLane.</i></p>

---

## 📌 Aim
To understand and perform pre-layout timing analysis of a digital design using **SKY130** technology — covering timing libraries, delay tables, synthesis, **OpenSTA**, and clock-tree concepts.

## 📖 Project Overview
This project focuses on **pre-layout timing analysis** using the SKY130 PDK. It explains how timing information is modelled using standard-cell libraries and delay tables, followed by synthesis and static timing analysis (STA) using OpenSTA. The project also covers:

- Ideal-clock timing analysis
- Clock Tree Synthesis (CTS)
- Clock buffering
- Crosstalk considerations
- Real-clock setup and hold analysis

Through these steps, the timing performance and possible timing violations of a digital design can be studied and improved before committing to physical layout.

## 🎯 Objectives
- [x] Convert grid information into routing tracks
- [x] Study SKY130 timing libraries across PVT corners (slow / typical / fast)
- [x] Understand LEF-based physical/library information for standard cells
- [x] Configure and run RTL-to-gate-level synthesis using Yosys
- [x] Perform ideal-clock setup timing analysis using OpenSTA
- [x] Understand clock jitter and clock uncertainty
- [x] Run placement and Clock Tree Synthesis (CTS)
- [x] Perform real-clock (post-CTS) setup and hold timing analysis
- [x] Compare pre-CTS and post-CTS slack values

## 🧰 Prerequisites & Setup

| Tool | Purpose | Reference |
|---|---|---|
| [SkyWater SKY130 Open PDK](https://github.com/google/skywater-pdk) | Liberty (`.lib`) and LEF technology files | `skywater-pdk` |
| [Yosys](https://yosyshq.net/yosys/) | RTL synthesis (Verilog → gate-level netlist) | `yosys -V` |
| [OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA) | Static timing analysis | `sta -version` |
| [OpenLane](https://github.com/The-OpenROAD-Project/OpenLane) | End-to-end RTL-to-GDS flow (placement, CTS) | `openlane --version` |

```bash
# Verify tool availability before starting
yosys -V
sta -version
```

> Ensure the SKY130 Liberty (`.lib`) and LEF files for the `slow` / `typical` / `fast` corners are available and correctly referenced before running synthesis or STA.

---

## Table of Contents
- [Grid-to-Track Conversion](#-grid-to-track-conversion)
- [Timing Libraries](#-timing-libraries)
- [Library Information (LEF)](#-library-information-lef)
- [Synthesis Configuration](#-synthesis-configuration)
- [Timing Analysis with Ideal Clocks](#-timing-analysis-with-ideal-clocks)
- [Clock Jitter and Uncertainty](#-clock-jitter-and-uncertainty)
- [OpenSTA Configuration](#-opensta-configuration)
- [Clock Tree Synthesis](#-clock-tree-synthesis)
- [Timing Analysis with Real Clocks](#-timing-analysis-with-real-clocks)
- [Results](#-results)
- [Learning Outcomes](#-learning-outcomes)
- [Conclusion](#-conclusion)
- [Repository Structure](#-repository-structure)
- [Tools Used](#-tools-used)
- [References](#-references)
- [License](#-license)

---

## 🔲 Grid-to-Track Conversion
Grid and track information is required for physical design and routing. The technology information is used to define the routing tracks that standard cells and interconnect must align to.

<p align="center">
  <img width="800" alt="sky130A_tech" src="https://github.com/user-attachments/assets/555de6e5-c7ee-46ff-8cb8-5d95053ae5ac" />
</p>

<p align="center"><sub><b>Figure 1.</b> SKY130 technology grid information used to derive routing tracks.</sub></p>

Track information can be inspected using the technology and LEF-related files.

<p align="center">
  <img width="800" alt="tracksinfo" src="https://github.com/user-attachments/assets/1c367663-b5ec-42ad-a0ef-a60a8b5940ea" />
</p>

<p align="center"><sub><b>Figure 2.</b> Routing track information derived from the SKY130 technology file.</sub></p>

---

## 📚 Timing Libraries
Static timing analysis relies on **Liberty (`.lib`)** timing libraries characterized across process, voltage, and temperature (PVT) corners. Real-clock timing analysis considers the actual clock network rather than an idealized clock source.

**Real-clock timing path**
```
Clock Source
   ↓
Clock Tree
   ↓
Launch Flip-Flop
   ↓
Combinational Logic
   ↓
Capture Flip-Flop
```

**PVT corner libraries used**

| Corner | Purpose |
|---|---|
| Fast | Best-case delay — used for hold-timing checks |
| Typical | Nominal operating condition |
| Slow | Worst-case delay — used for setup-timing checks |

<p align="center">
  <img width="700" alt="fastlib" src="https://github.com/user-attachments/assets/a7c436da-908c-4824-988d-243257384763" />
</p>
<p align="center"><sub><b>Figure 3.</b> Fast-corner timing library.</sub></p>

<p align="center">
  <img width="700" alt="typicallib" src="https://github.com/user-attachments/assets/e8243854-03e9-4a4e-b6bd-c73dbb590657" />
</p>
<p align="center"><sub><b>Figure 4.</b> Typical-corner timing library.</sub></p>

<p align="center">
  <img width="700" alt="slowlib" src="https://github.com/user-attachments/assets/1e8de6d6-94f7-4855-b5c8-20368255248c" />
</p>
<p align="center"><sub><b>Figure 5.</b> Slow-corner timing library.</sub></p>

---

## 📄 Library Information (LEF)
LEF files contain the **physical** information about standard cells (pin locations, cell dimensions, routing obstructions) and are used during physical design and place-and-route.

<p align="center">
  <img width="800" alt="leffff" src="https://github.com/user-attachments/assets/e94b9bf4-5385-4f52-adac-ed11592c8dc4" />
</p>

<p align="center"><sub><b>Figure 6.</b> SKY130 standard-cell LEF information.</sub></p>

---

## ⚙️ Synthesis Configuration
Synthesis converts the RTL design into a gate-level netlist using the selected standard-cell library.

```tcl
yosys
read_verilog <design>.v
read_liberty -lib <library>.lib
synth -top <top_module>
stat
```

<p align="center">
  <img width="800" alt="analy" src="https://github.com/user-attachments/assets/5a09832a-5407-48de-9e20-c7ed61b3ba6e" />
</p>

<p align="center"><sub><b>Figure 7.</b> Synthesis run and cell statistics.</sub></p>

The synthesized design can then be inspected to verify the generated cells and design statistics.

<p align="center">
  <img width="800" alt="analy2" src="https://github.com/user-attachments/assets/fa180794-3afd-4118-815e-843f44c15a31" />
</p>

<p align="center"><sub><b>Figure 8.</b> Post-synthesis design inspection.</sub></p>

---

## ⏱️ Timing Analysis with Ideal Clocks

### Setup Timing Analysis
Setup timing verifies whether data reaches the destination flip-flop **within the required time before the clock edge**.

```tcl
read_liberty <library>.lib
read_verilog <design>.v
link_design <top_module>
read_sdc <constraints>.sdc
report_checks -path_delay max
```

## 🕰️ Clock Jitter and Uncertainty
- **Clock jitter** represents variation in clock arrival time from cycle to cycle.
- **Clock uncertainty** provides a timing margin in STA to account for such variations, along with other non-idealities (e.g. OCV, crosstalk margins).

## 🧮 OpenSTA Configuration
```tcl
read_liberty <library>.lib
read_verilog <design>.v
link_design <top_module>
read_sdc <constraints>.sdc
report_checks
```

---

## 🌳 Clock Tree Synthesis

### Placement
Before CTS, the design undergoes **placement**, where standard cells are positioned within the floorplan.

<p align="center">
  <img width="800" alt="vsdinvlayout1" src="https://github.com/user-attachments/assets/46f604af-3b1a-4f02-b5e7-7fef8421299f" />
</p>
<p align="center"><sub><b>Figure 9.</b> Standard-cell placement, overview.</sub></p>

<p align="center">
  <img width="800" alt="vsdinvlayout2" src="https://github.com/user-attachments/assets/224aaece-c890-4de4-91c7-bf5bea00f3b4" />
</p>
<p align="center"><sub><b>Figure 10.</b> Placement inspected in greater detail.</sub></p>

<p align="center">
  <img width="800" alt="vsdinvlayout3" src="https://github.com/user-attachments/assets/1356f4df-794e-4cd8-9b5f-3dcf64538b64" />
</p>
<p align="center"><sub><b>Figure 11.</b> Enlarged placement view.</sub></p>

### Running CTS
<p align="center">
  <img width="800" alt="runcts" src="https://github.com/user-attachments/assets/da85fbaa-2032-41ff-96dd-06f3c201c679" />
</p>
<p align="center"><sub><b>Figure 12.</b> Clock Tree Synthesis run.</sub></p>

---

## 🔁 Timing Analysis with Real Clocks

### Pre-STA (Post-CTS) Checks
<p align="center">
  <img width="800" alt="presta2" src="https://github.com/user-attachments/assets/99c9ac3f-6066-4af0-ba6b-a56db93c2f24" />
</p>
<p align="center"><sub><b>Figure 13.</b> Pre-STA report, overview.</sub></p>

<p align="center">
  <img width="800" alt="presta_sta" src="https://github.com/user-attachments/assets/45986cfa-f1e2-49a5-a1a4-8c46883c224e" />
</p>
<p align="center"><sub><b>Figure 14.</b> Pre-STA timing report detail.</sub></p>

### Slack Comparison (Before vs. After CTS)
<p align="center">
  <img width="700" alt="oldslack" src="https://github.com/user-attachments/assets/82e86202-118e-4daf-8c31-5670b565b1b6" />
  <img width="700" alt="newslack" src="https://github.com/user-attachments/assets/8dafd94a-d91a-4e56-8fba-afa0692a035a" /><br/>
  <img width="700" alt="oldslack1" src="https://github.com/user-attachments/assets/aedb6295-0013-433d-af56-100af706b888" />
  <img width="700" alt="newslack1" src="https://github.com/user-attachments/assets/bcae774a-9d4f-4a34-806b-e4aad7d1c41d" />
</p>
<p align="center"><sub><b>Figure 15.</b> Setup/hold slack values before (old) and after (new) clock-tree synthesis.</sub></p>

---



## 🎓 Learning Outcomes
By completing this lab, the following practical skills were developed:
- Reading SKY130 Liberty and LEF files and relating them to PVT corners.
- Running RTL-to-gate-level synthesis in Yosys and interpreting cell statistics.
- Writing OpenSTA command sequences to link a design and report timing checks.
- Distinguishing ideal-clock STA from real-clock (post-CTS) STA.
- Understanding the role of clock jitter and clock uncertainty in setup/hold margins.
- Reading placement and Clock Tree Synthesis output, and comparing pre-/post-CTS slack.

## ✅ Conclusion
The SKY130 pre-layout timing flow was studied successfully. Timing libraries and delay tables were understood, synthesis was configured and run, and setup/hold timing analysis was performed — first conceptually with ideal clocks, then with real clocks — using OpenSTA. Clock Tree Synthesis and the resulting shift in setup/hold slack were also studied, along with the role of clock jitter and clock uncertainty in accounting for real-world timing margins. This module builds a practical understanding of how timing information is generated and used both before and after clock-tree implementation, helping to identify and resolve timing problems early in the design flow.

## 📁 Repository Structure
```
.
├── libs/                    # SKY130 slow/typical/fast Liberty libraries + LEF
├── rtl/                     # RTL source (Verilog)
├── synthesis/                # Yosys scripts, synthesis logs, stat reports
├── sta/                      # OpenSTA scripts, .sdc constraints, timing reports
├── cts/                      # OpenLane placement + CTS run outputs
├── images/                   # Screenshots referenced in this README
└── README.md
```

## 🛠️ Tools Used
- **SkyWater SKY130** — open-source PDK (Liberty + LEF)
- **Yosys** — RTL synthesis
- **OpenSTA** — static timing analysis
- **OpenLane** — placement and Clock Tree Synthesis

## 📚 References
- SkyWater SKY130 Open PDK documentation
- OpenSTA user guide (The OpenROAD Project)
- OpenLane documentation (The OpenROAD Project)
- Yosys manual


---

<p align="center"><sub>⭐ If you found this timing-analysis walkthrough useful, consider starring the repository.</sub></p>

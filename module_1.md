# OpenLane Physical Design – PicoRV32A

This project is a basic study of the **RTL-to-GDSII design flow** using **PicoRV32A, OpenLane, and the Sky130 PDK**.

The project includes important stages such as synthesis, floorplanning, power planning, placement, Clock Tree Synthesis (CTS), routing, Static Timing Analysis (STA), and signoff.

---

## 1. Project Overview

ASIC physical design is the process of transforming a digital circuit description into an actual physical layout of a chip.

In this project, the **PicoRV32A RISC-V processor** is used to understand the different stages involved in the complete physical design flow.

### Flow

```text
RTL
 ↓
Synthesis
 ↓
Floorplanning
 ↓
Placement
 ↓
CTS
 ↓
Routing
 ↓
Signoff
 ↓
GDSII
```

---

## 2. PicoRV32A

PicoRV32A is a compact **RISC-V processor** designed using RTL code.

RTL describes the functionality and operation of a digital circuit.

The design includes components such as:

* Logic gates
* Flip-flops
* Multiplexers
* Registers
* Control logic

The RTL design is given as the input to the synthesis stage.

---

## 3. PDK

**PDK stands for Process Design Kit.**

A PDK contains the technology-related information required for designing and manufacturing an integrated circuit.

It provides information such as:

* Standard cell details
* Technology layers
* Design rules
* Timing data
* Physical information

In this project, the **Sky130 PDK** is used.

---

## 4. OpenLane

**OpenLane is an open-source flow used to convert RTL designs into GDSII layouts.**

It combines different tools and performs the major steps required in ASIC physical design.
<img width="1280" height="768" alt="open2" src="https://github.com/user-attachments/assets/b5b45f78-20b7-46ff-a185-f726347a92d7" />

<img width="1280" height="768" alt="open1" src="https://github.com/user-attachments/assets/917ad504-e8ce-4a00-aab9-8478f02bab57" />


### Simple OpenLane Flow

```text
RTL + PDK
   ↓
Synthesis
   ↓
Floorplanning
   ↓
Placement
   ↓
CTS
   ↓
Routing
   ↓
Signoff
   ↓
GDSII
```

OpenLane simplifies the ASIC physical design process by using various open-source tools.

---

## 5. Synthesis

**Synthesis is the process of converting RTL code into a gate-level netlist.**

During synthesis, the RTL design is analyzed and mapped into standard cells available in the technology library.

```text
RTL
 ↓
Synthesis
 ↓
Gate-Level Netlist
```

The synthesized circuit may contain:

* AND gates
* OR gates
* NAND gates
* NOR gates
* Buffers
* Inverters
* Multiplexers
* Flip-flops

### Synthesis Result

The synthesis result shows how the RTL design has been converted into a gate-level implementation.


<img width="1280" height="768" alt="synthesissss" src="https://github.com/user-attachments/assets/78805a58-2988-4c13-ab9a-1b3ce818fb50" />



---

## 6. Netlist

A **netlist represents the cells present in a circuit and the connections between them**.

After the synthesis process, the RTL description is transformed into a gate-level netlist.

```text
RTL
 ↓
Standard Cells
 ↓
Connections
 ↓
Gate-Level Netlist
```

The PicoRV32A design contains a large number of standard cells connected together.

### Generated Netlist

The generated netlist provides the gate-level representation of the design.
<img width="1280" height="768" alt="picorvsynthesisinform" src="https://github.com/user-attachments/assets/edf3b941-1013-4fbe-95e6-739237045284" />


---

## 7. Floorplanning

**Floorplanning determines the initial physical organization of the chip.**

It mainly defines:

* Chip dimensions
* Core dimensions
* Locations of I/O pins
* Available area for placing cells

A proper floorplan helps improve the placement and routing stages.
<img width="1280" height="768" alt="floorplan" src="https://github.com/user-attachments/assets/144d236c-2e47-428f-bfa6-dfae0ef5b4f9" />


---

## 8. Power Planning

**Power planning creates the power distribution network for the chip.**

It includes components such as:

* Power rings
* Power straps
* Power rails
* VDD
* VSS

The main objective is to supply power correctly and efficiently to all the cells in the design.

---

## 9. Placement

**Placement determines the physical locations of standard cells inside the design area.**

Placement is mainly divided into two stages.

### Global Placement

Global placement determines approximate positions for the cells.

### Detailed Placement

Detailed placement adjusts the cells into valid and legal locations.

Good placement helps to:

* Minimize wire length
* Reduce congestion
* Improve timing performance
* Simplify the routing process
  <img width="1280" height="768" alt="placement" src="https://github.com/user-attachments/assets/ed68991e-0842-469d-bd96-5d4432e23090" />
<img width="1280" height="768" alt="area" src="https://github.com/user-attachments/assets/70232da1-59b8-4524-9f15-e151c31b710e" />


---

## 10. Clock Tree Synthesis

**CTS stands for Clock Tree Synthesis.**

CTS is used to create and distribute the clock network to sequential elements such as flip-flops.

Buffers are inserted to ensure proper clock distribution throughout the design.

### Main Goal

The primary goal of CTS is to ensure that the clock signal reaches all flip-flops at the appropriate time.

CTS also attempts to minimize **clock skew**.

```text
        Clock
          |
        Buffer
       /  |  \
     FF1 FF2 FF3
```

---

## 11. Routing

**Routing creates the physical connections between all placed cells using metal layers.**

Routing generally consists of two stages.

### Global Routing

Global routing identifies the approximate paths required for the connections.

### Detailed Routing

Detailed routing generates the actual metal connections between cells.

All routing must follow the technology design rules.

### RTL to GDSII Flow

The complete flow gradually converts the RTL design into the final physical GDSII layout.

---

## 12. STA

**STA stands for Static Timing Analysis.**

STA verifies whether the circuit satisfies the required timing constraints.

It checks parameters such as:

* Cell delay
* Net delay
* Clock delay
* Setup time
* Hold time
* Slew
  


### Slack

Slack indicates whether the timing requirements of the design are satisfied.

```text
Positive Slack → Timing is satisfied

Negative Slack → Timing violation
```

**OpenSTA** is used to perform timing analysis.

### STA Report

The STA report provides information about the timing performance of the design.
<img width="1280" height="768" alt="sta" src="https://github.com/user-attachments/assets/6a5abdb2-ccec-49cf-8481-dcb2f18062e8" />


---

## 13. Signoff

**Signoff is the final verification stage in the physical design flow.**

Before generating the final layout, the design undergoes several important checks.

### DRC

**Design Rule Check**

DRC verifies whether the layout follows the manufacturing and technology rules.

### LVS

**Layout Versus Schematic**

LVS checks whether the physical layout matches the intended circuit design.

### Timing

Timing verification ensures that the design meets the required timing constraints.

After all checks are successfully completed, the final design can be generated in the form of **GDSII**.

---

## 14. OpenLane Commands

The OpenLane flow can be launched using:

```text
./flow.tcl -interactive
```

Some of the commands used during the flow include:

```text
package require openlane 0.9
prep -design picorv32a
run_synthesis
```

These commands are used to initialize the OpenLane environment, prepare the design, and perform synthesis.

The commands may vary slightly depending on the version of OpenLane being used.

---

## 15. Results and Learning

### Synthesis Statistics

The recorded synthesis result for the PicoRV32A design is shown below:

| Parameter        |  Value |
| ---------------- | -----: |
| Total Wires      | 22,926 |
| Wire Bits        | 25,811 |
| Public Wires     |  1,162 |
| Public Wire Bits |  1,972 |
| Memories         |      0 |
| Memory Bits      |      0 |
| Processes        |      0 |
| Total Cells      | 18,036 |
| Flip-Flops       |  1,613 |
### Design Statistics

The design statistics provide an overall summary of the synthesized PicoRV32A design.
<img width="1280" height="768" alt="runsysnthesis" src="https://github.com/user-attachments/assets/8c0112df-1818-43ec-84cc-dc52277f4eb0" />



### Flip-Flop Ratio

The flip-flop ratio represents the percentage of flip-flops compared with the total number of cells.

### Formula

Flip-Flop Ratio = (Flip-Flops / Total Cells) × 100

Using your values:

$$ \text{Flip-Flop Ratio}=\left(\frac{1613}{18036}\right)\times100 $$
Flip-Flop Ratio≈8.94%
(1613 / 18036) × 100
= 8.94%

Therefore,

Flip-Flop Ratio ≈ 8.94%

---

### Complete Flow

```text
RTL
 ↓
Synthesis
 ↓
Floorplanning
 ↓
Power Planning
 ↓
Placement
 ↓
Clock Tree Synthesis
 ↓
Routing
 ↓
STA
 ↓
Physical Verification
 ↓
Signoff
 ↓
GDSII
```

---

### Tools Used

| Tool      | Purpose               |
| --------- | --------------------- |
| PicoRV32A | RISC-V processor      |
| OpenLane  | Physical design flow  |
| Yosys     | Synthesis             |
| OpenSTA   | Timing analysis       |
| Sky130    | Technology / PDK      |
| GDSII     | Final physical layout |

---

## What I Learned

Through this project, I gained an understanding of the basic **ASIC physical design flow**.

I learned how a digital design is transformed from RTL code into a physical chip layout.

The overall concept can be represented as:

```text
RTL
 ↓
Netlist
 ↓
Physical Design
 ↓
Verification
 ↓
GDSII
```

This project helped me understand the following concepts:

* RTL
* PDK
* Synthesis
* Netlist
* Floorplanning
* Power planning
* Placement
* Clock Tree Synthesis
* Routing
* STA
* Physical verification
* Signoff

Overall, this project provided practical knowledge of the ASIC physical design process using **open-source tools and Sky130 technology**.

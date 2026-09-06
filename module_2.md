## PicoRV32A Physical Layout using Magic VLSI
## 1. Introduction

This project presents the physical layout inspection of the PicoRV32A RISC-V processor using Magic VLSI and the SKY130A technology.

The screenshots show the PicoRV32A design at different zoom levels in the Magic VLSI layout environment. The layout contains standard cells, physical cells, signal connections, pins, and different technology layers.

The main purpose of this activity is to understand how a synthesized digital circuit is represented physically in an ASIC layout.

## 2. Objective

The objectives of this practical are:

To understand the basics of physical layout.
To inspect the PicoRV32A layout using Magic VLSI.
To understand standard-cell placement.
To identify different cells and signal names in the layout.
To understand the use of different physical layers.
To observe the layout at different zoom levels.
To understand the relationship between the gate-level design and physical implementation.
To become familiar with the Magic VLSI layout environment.
## 3. Tools and Technology Used
Tool / Technology	Purpose
PicoRV32A	RISC-V processor design
Magic VLSI	Layout viewing and inspection
SKY130A	CMOS technology / PDK
Standard Cell Library	Implementation of digital logic
DRC	Design Rule Checking
LVS	Layout Versus Schematic verification
GDSII	Physical layout data format
## 4. PicoRV32A

PicoRV32A is a compact 32-bit RISC-V processor design.

The processor is initially described at the RTL level. During the digital implementation process, the RTL is converted into a gate-level representation using standard cells.

The physical implementation contains cells such as:

Flip-flops
Multiplexers
Buffers
Inverters
NOR gates
AND gates
Other logic cells

These cells can be seen physically in the Magic VLSI layout.

## 5. SKY130A Technology

SKY130A is a 130 nm CMOS technology used for implementing integrated circuits.

The technology provides the required information for physical design, including:

Technology layers
Standard cells
Design rules
Physical dimensions
Cell layouts
Electrical and timing information

Magic VLSI uses this technology information to display the physical layout correctly.

## 6. Magic VLSI Layout

Magic VLSI is used to view and inspect the physical implementation of the PicoRV32A design.



PicoRV32A design
Standard-cell instances
Physical cells
Signal labels
Metal and other technology layers
Cell boundaries
Power-related structures
Layout geometry

The top bar also shows that the design is being viewed using the SKY130A technology.

## 7. Layout Observations
Figure 1 – Complete Layout at High-Level View

<img width="1280" height="768" alt="layout1" src="https://github.com/user-attachments/assets/ee365cde-8a01-46b4-9493-a041923f7041" />


At this level, the complete design appears as a large rectangular region containing many repeated vertical structures.

Because the design contains a large number of physical geometries, individual cells cannot be easily identified at this zoom level.

Observation
Overall layout boundary can be observed.
The design occupies a rectangular physical region.
A large number of repeated structures are visible.
This view gives an idea of the overall size and density of the layout.
Figure 2 – Zoomed Layout with Signal and Cell Names

The second screenshot shows a zoomed portion of the layout.
<img width="1280" height="768" alt="layout2" src="https://github.com/user-attachments/assets/52bb4e05-ce29-44c4-a216-cadbc86ee7ad" />


Some signal names such as:

pcpi_insn[30]
eoi[24]

can be observed.

The layout also contains physical cell names such as:

PHY_252
PHY_250
PHY_248
PHY_246
PHY_244
PHY_242

The top-level cell picorv32a is also visible.

Observation

This view helps identify the relationship between signal names and physical structures in the layout.

Figure 3 – PicoRV32A Internal Layout

The third screenshot shows another zoomed portion of the PicoRV32A layout.
<img width="1280" height="768" alt="layout3" src="https://github.com/user-attachments/assets/6312993b-0ec4-4cce-b3d5-9b80033edca6" />


A signal named:

trace_data[26]

is visible.

The screenshot also shows several rectangular physical regions and standard-cell structures.

Observation

At this zoom level, the internal organization of the design becomes easier to observe. Different physical regions can be distinguished from one another.

Figure 4 – Complete PicoRV32A Layout

The fourth screenshot provides another high-level view of the complete PicoRV32A layout.
<img width="1280" height="768" alt="layout4" src="https://github.com/user-attachments/assets/e4f75b9c-e961-4712-9b6a-d26dd7835d10" />


The name:

picorv32a

can be seen near the center of the layout.

The layout appears as a dense rectangular block containing many repeated physical structures.

Observation

This view demonstrates that a processor design contains a large number of physical geometries after implementation.





## 8. Standard Cells in the Layout

The screenshots contain many standard-cell instances from the SKY130 standard-cell library.

Examples include:

sky130_fd_sc_hd__dff...
sky130_fd_sc_hd__mux...
sky130_fd_sc_hd__buf...
sky130_fd_sc_hd__inv...
sky130_fd_sc_hd__nor...

The standard cells implement the logical functions required by the PicoRV32A processor.

For example:

DFF – Flip-flop
MUX – Multiplexer
BUF – Buffer
INV – Inverter
NOR – NOR logic

These cells are physically placed and connected to create the complete processor.

## 9. Physical Cell Names

The screenshots also contain names such as:

PHY_252
PHY_250
PHY_248
PHY_246
PHY_244
PHY_242

These names identify physical cells or physical structures present in the layout.

They demonstrate that the physical implementation contains more than just the logical standard cells.

## 10. Signal Names

Some signal names can be observed in the screenshots.

Examples are:

pcpi_insn[30]
eoi[24]
trace_data[26]

These labels help identify specific signals in the design.

A signal connects different circuit elements and represents electrical connectivity in the processor.

## 11. Placement

Placement is the process of deciding where standard cells are physically located inside the chip area.

The screenshots show standard cells arranged in organized rows.

A simplified flow is:

Standard Cells
      ↓
Global Placement
      ↓
Detailed Placement
      ↓
Placed Design

Figure 5 – Dense Physical Layout

The fifth screenshot shows the PicoRV32A layout with a large number of visible cells and physical geometries.

Several cell names can be observed, including names containing:

sky130_fd_sc_hd__

These are SKY130 standard-cell library instances.

Examples visible in the layout include cells associated with:

dff
mux
buf
inv
nor
Observation

<img width="1280" height="768" alt="placeembt" src="https://github.com/user-attachments/assets/3ec77e5b-5bb6-4aa3-b5e1-41ddab5672cf" />

<img width="1280" height="768" alt="placement2" src="https://github.com/user-attachments/assets/5bebf90f-a376-4d89-8b90-b3c3fd5b71a5" />
Figure 7 – Detailed Cell and Layer View

The seventh screenshot provides a closer view of the physical layout.

The layout shows:

Standard-cell boundaries
Cell names
Physical layers
Vertical structures
Horizontal structures
Connections between cells

The different colors and patterns represent different types of physical layout layers.

<img width="1280" height="768" alt="placement3" src="https://github.com/user-attachments/assets/7ddd5dcc-7626-463b-b33c-9ccbd575ea08" />
<img width="1280" height="768" alt="placement5" src="https://github.com/user-attachments/assets/6e7fd077-3903-48d1-ae91-d1215729249a" />



Good placement helps to:

Reduce wire length
Reduce congestion
Improve timing
Improve area utilization
Make routing easier

The repeated cell structures visible in the screenshots indicate the physical placement of standard cells.

## 12. Routing

Routing creates physical connections between the placed cells.

Different metal layers are used to connect signals.

The routing process can be represented as:

Placed Cells
      ↓
Global Routing
      ↓
Detailed Routing
      ↓
Physical Connections

The physical wires and layer geometries visible in the zoomed screenshots represent the connections between different parts of the design.

## 13. Layout Layers

An IC layout consists of multiple technology layers.

These layers are used for:

Device formation
Contacts
Interconnections
Metal routing
Power connections

Magic VLSI displays different layers using different colors and patterns.

The layer palette on the right side of the Magic window allows the user to identify and select different layers.

## 14. Power and Ground

A physical design also requires proper power distribution.

The main supply connections are:

VDD → Power
VSS → Ground

Power structures distribute the required supply to the standard cells.

A properly designed power network helps ensure reliable operation of the circuit.

## 15. Layout Hierarchy

The top-level design shown in the screenshots is:

picorv32a

Inside the top-level design, many standard-cell and physical-cell instances are present.

The hierarchy can be represented as:

picorv32a
    |
    +── Standard Cells
    |
    +── Physical Cells
    |
    +── Signal Connections
    |
    +── Power Structures

This hierarchical organization helps in understanding a large physical design.



## 16. Physical Verification

After creating the physical layout, verification is required.

DRC – Design Rule Check

DRC checks whether the layout follows the manufacturing rules of the selected technology.

It checks rules such as:

Minimum width
Minimum spacing
Via rules
Layer restrictions
Enclosure requirements
LVS – Layout Versus Schematic

LVS checks whether the physical layout connectivity matches the intended circuit/netlist.

The basic verification flow is:

Physical Layout
       ↓
      DRC
       ↓
      LVS
       ↓
Verified Layout
## 17. RTL to Physical Layout

The overall relationship between the digital design and the physical layout can be represented as:

RTL
 ↓
Synthesis
 ↓
Gate-Level Netlist
 ↓
Standard Cells
 ↓
Placement
 ↓
Routing
 ↓
Physical Layout
 ↓
DRC / LVS
 ↓
GDSII

The screenshots provided in this practical mainly demonstrate the physical layout inspection stage.

## 18. Observations

From the given PicoRV32A Magic VLSI screenshots, the following observations were made:

The complete PicoRV32A design occupies a rectangular physical area.
The layout contains a large number of standard-cell instances.
Standard cells are arranged in organized physical regions.
SKY130 standard-cell names can be observed in the layout.
Physical cell names such as PHY_252, PHY_250, and others are visible.
Signal names such as pcpi_insn[30], eoi[24], and trace_data[26] are visible.
Different physical layers are represented using different colors and patterns.
At higher zoom levels, individual cells and their connections can be inspected.
At lower zoom levels, the complete physical area and overall design density can be observed.
The layout demonstrates the conversion of synthesized digital logic into physical geometry.

## Conclusion

The PicoRV32A physical layout inspection using Magic VLSI provided a basic understanding of how a digital processor is converted into a physical integrated-circuit layout.

The provided screenshots show the design at different zoom levels and demonstrate the presence of standard cells, physical cells, signal labels, routing structures, and different technology layers.

By examining the layout, the relationship between the gate-level design and its physical implementation can be understood.

This practical provides a foundation for learning advanced ASIC physical-design concepts such as floorplanning, placement, Clock Tree Synthesis, routing, Static Timing Analysis, DRC, LVS, and final GDSII generation.

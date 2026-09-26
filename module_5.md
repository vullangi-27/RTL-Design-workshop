# Power Distribution Network (PDN) & Detailed Routing (TritonRoute)

## Aim
To implement the Power Distribution Network (PDN), perform detailed routing using TritonRoute, and carry out Design Rule Checking (DRC) layout verification for the picorv32a RISC-V core using the Sky130 PDK within the OpenLANE flow.

## Objectives
- Construct power and ground rings, straps, and rails (`gen_pdn`) across the layout following Clock Tree Synthesis (CTS).
- Perform global and detailed routing using OpenROAD and TritonRoute, ensuring no setup or hold timing violations.
- Stream out the GDSII file using Magic and verify physical design rules to achieve zero DRC errors.
- Resolve state execution dependencies and disk space constraints encountered during interactive flow runs.

## Procedure

### 1. Environment Setup & Prerequisites
The OpenLANE interactive Tcl shell was launched and the target design initialized:

\`\`\`tcl
package require openlane 0.9
prep -design picorv32a -tag RUN_MOD5_CLEAN -overwrite
\`\`\`

The prerequisite physical design stages were then executed sequentially:
**Synthesis → Floorplan → Placement → Clock Tree Synthesis (CTS)**

### 2. Power Distribution Network (PDN) Generation
Power (`VDD`) and ground (`VSS`) net environment variables were explicitly configured to prevent Tcl state corruption. The PDN generation step was then run to build the power grid rings and standard cell rails:

\`\`\`tcl
set ::env(VDD_NET) "VDD"
set ::env(GND_NET) "VSS"
gen_pdn
\`\`\`

<img width="1280" height="767" alt="PDN structure generated across the floorplan" src="https://github.com/user-attachments/assets/12e3af38-b4b7-4203-bea3-7dcc21400078" />

*Fig. 1 — Power distribution network generated across the floorplan.*

<img width="1280" height="767" alt="Power and ground rails connected to standard cells" src="https://github.com/user-attachments/assets/86393601-2599-45da-8b37-827c87b1f5c7" />

*Fig. 2 — Power/ground rails connected to standard cell rows.*

### 3. Detailed Routing
Global and detailed routing were carried out using TritonRoute to route all design signals across the Sky130 metal stack:

\`\`\`tcl
run_routing
\`\`\`

### 4. GDSII Streaming & DRC Verification
Legacy run logs and temporary build directories were cleared (`rm -rf`) to resolve VM disk space limitations (0 bytes remaining). The layout was then streamed out to GDSII format and subjected to full physical DRC verification using Magic:

\`\`\`tcl
run_magic
run_magic_drc
\`\`\`

<img width="1280" height="767" alt="Streamed-out GDSII layout viewed in Magic" src="https://github.com/user-attachments/assets/8536be3d-8b58-4756-8613-1206b286a171" />

*Fig. 3 — Streamed-out GDSII layout viewed in Magic.*

<img width="1280" height="767" alt="Magic DRC verification in progress" src="https://github.com/user-attachments/assets/a90eb03a-6826-4667-a7b9-ca0d53ee0554" />

*Fig. 4 — DRC verification executing in Magic.*

<img width="1280" height="767" alt="Magic DRC results showing zero violations" src="https://github.com/user-attachments/assets/7e66377a-7178-4fbf-b815-4af99c37ad9a" />

*Fig. 5 — DRC report confirming zero rule violations.*

### 5. Report Generation
Final runtime and overall summary reports were generated to close out the module:

\`\`\`tcl
calc_total_runtime
generate_final_summary_report
\`\`\`

<img width="1280" height="767" alt="Final summary report output" src="https://github.com/user-attachments/assets/0276d402-3260-433c-ba7f-d62308821f01" />

*Fig. 6 — Final summary report generated for the run.*

## Results
| Metric | Outcome |
|---|---|
| **PDN Status** | Power grid successfully synthesized and connected to standard cell supply pins |
| **Routing Status** | TritonRoute completed detailed routing with no unrouted nets remaining |
| **Layout Rule Checks** | Magic DRC verification confirmed 0 DRC violations on the streamed GDSII layout |
| **Report Artifacts** | Saved under `/openLANE_flow/designs/picorv32a/runs/RUN_MOD5_CLEAN/reports/final_summary_report.csv` |

## Conclusion
Module 5 of the RTL2GDS flow was completed successfully. The picorv32a design was fully routed, satisfied all layout constraints, and passed complete physical verification with zero DRC violations — producing a clean, tape-out-ready GDSII file.

# Day 4: Gate-Level Simulation (GLS), Blocking vs. Non-Blocking in Verilog, and Synthesis-Simulation Mismatch

Welcome to Day 4 of the RTL Workshop. Today's session focuses on three essential topics in digital design:

- Gate-Level Simulation (GLS)
- Blocking vs. Non-Blocking Assignments in Verilog
- Synthesis-Simulation Mismatch

You will explore both the underlying theory and its practical implications, reinforced through hands-on labs.

---

## Table of Contents

1. [Gate-Level Simulation (GLS)](#1-gate-level-simulation-gls)
2. [Synthesis-Simulation Mismatch](#2-synthesis-simulation-mismatch)
3. [Blocking vs. Non-Blocking Assignments in Verilog](#3-blocking-vs-non-blocking-assignments-in-verilog)
   - [3.1 Blocking Statements](#31-blocking-statements-)
   - [3.2 Non-Blocking Statements](#32-non-blocking-statements-)
   - [3.3 Comparison Table](#33-comparison-table)
4. [Labs](#4-labs)
5. [Summary](#5-summary)

---

## 1. Gate-Level Simulation (GLS)

GLS, or Gate-Level Simulation, is a critical verification step in the VLSI design flow in which the synthesized gate-level netlist of a digital circuit is simulated to validate:

- Functional correctness
- Timing behavior
- Power estimates
- Test structures (e.g., scan chains for DFT)

**Why Perform GLS?**
- **Synthesis Validation:** Confirms that the synthesis tool has faithfully translated the RTL description into an equivalent gate-level implementation.
- **Timing Verification:** Simulates the design using realistic delay values (annotated from SDF files), allowing timing violations such as setup and hold errors to be detected.
- **Testability:** Verifies that scan chains and other design-for-test (DFT) structures function correctly after synthesis.

**When is GLS Performed?**
- **After synthesis:** Once the RTL has been converted into a gate-level netlist.
- **Before physical design:** To catch functional and timing issues early, prior to layout.

**Types of GLS**
- **Functional GLS:** A logic-only simulation, typically run with zero or unit delays.
- **Timing GLS:** Uses annotated timing data to verify real-world timing behavior.

---

## 2. Synthesis-Simulation Mismatch

A synthesis-simulation mismatch occurs when the simulation results of the pre-synthesis RTL differ from the simulation results of the post-synthesis gate-level netlist, or from actual hardware behavior. Common causes include:

- **Non-synthesizable constructs:** Use of delays, `initial` blocks, or other RTL constructs that are not supported by synthesis tools.
- **Incomplete or ambiguous coding:** For example, missing `else` clauses or improperly specified sensitivity lists.
- **Tool interpretation differences:** Simulation and synthesis tools may interpret ambiguous RTL constructs differently, leading to divergent behavior.

> **Key Point:** Always write synthesizable, unambiguous RTL and follow good coding practices to minimize the risk of mismatches.

---

## 3. Blocking vs. Non-Blocking Assignments in Verilog

Verilog provides two types of procedural assignments, each suited to a different style of hardware description.

### 3.1 Blocking Statements (`=`)

- **Syntax:** `=`
- **Execution:** Sequential; each statement executes immediately, in the order it appears.
- **Suitable for:** Combinational logic (e.g., `always @(*)`).

**Example:**
```verilog
always @(*) y = a & b;
```

### 3.2 Non-Blocking Statements (`<=`)

- **Syntax:** `<=`
- **Execution:** Scheduled; all right-hand-side values are evaluated first, and updates are applied concurrently at the end of the current time step.
- **Suitable for:** Sequential logic (e.g., `always @(posedge clk)`).

**Example:**
```verilog
always @(posedge clk) q <= d;
```

### 3.3 Comparison Table

| Blocking (`=`) | Non-Blocking (`<=`) |
|---|---|
| Uses the `=` operator | Uses the `<=` operator |
| Sequential; executes immediately | Concurrent; scheduled for the end of the time step |
| Updates occur instantly, in code order | Updates are applied only after the time step completes |
| Suited to combinational logic and temporary variables | Suited to sequential logic, registers, and flip-flops |
| Infers combinational logic (gates) | Infers sequential logic (flip-flops) |

---

## 4. Labs

### Lab 1: Ternary Operator MUX

Verilog code for a simple 2-to-1 multiplexer implemented using a ternary operator:

```verilog
module ternary_operator_mux (input i0, input i1, input sel, output y);
  assign y = sel ? i1 : i0;
endmodule
```

**Functionality:** `y = i1` when `sel = 1`; otherwise, `y = i0`.

<img width="1280" height="800" alt="ternary" src="https://github.com/user-attachments/assets/43fc8097-9de7-41cd-9ace-75f33f713b82" />


---

### Lab 2: Synthesis Using Yosys

Synthesize the multiplexer from Lab 1 using Yosys, following the standard Yosys synthesis flow.

<img width="1280" height="800" alt="turnary1" src="https://github.com/user-attachments/assets/3f464e06-ccad-413e-b0c1-2ccc151113bf" />



---

### Lab 3: Gate-Level Simulation (GLS) of the MUX

Run GLS on the synthesized multiplexer using a command similar to the following (adjust paths as needed):

```shell
iverilog /path/to/primitives.v /path/to/sky130_fd_sc_hd.v ternary_operator_mux.v testbench.v
```
<img width="1280" height="800" alt="ternary" src="https://github.com/user-attachments/assets/43fc8097-9de7-41cd-9ace-75f33f713b82" />



---

### Lab 4: Bad MUX Example (Common Pitfalls)

Verilog code containing intentional design issues:

```verilog
module bad_mux (input i0, input i1, input sel, output reg y);
  always @ (sel) begin
    if (sel)
      y <= i1;
    else
      y <= i0;
  end
endmodule
```

**Issues identified:**
- **Incomplete sensitivity list:** The sensitivity list should include `i0` and `i1` in addition to `sel`.
- **Non-blocking assignment in combinational logic:** Blocking assignments (`=`) should be used instead.

**Corrected version:**
```verilog
always @ (*) begin
  if (sel)
    y = i1;
  else
    y = i0;
end
```

<img width="1280" height="800" alt="badmux" src="https://github.com/user-attachments/assets/aa78335b-f4fb-4700-98a8-a92e3cb997de" />


---

### Lab 5: GLS of the Bad MUX

Perform GLS on `bad_mux`. Due to the issues identified above, simulation mismatches or synthesis warnings should be expected.

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_50_15" src="https://github.com/user-attachments/assets/3229607a-ab87-4d4c-9ee3-d8c636f2d867" />


---

### Lab 6: Blocking Assignment Caveat

Verilog code:

```verilog
module blocking_caveat (input a, input b, input c, output reg d);
  reg x;
  always @ (*) begin
    d = x & c;
    x = a | b;
  end
endmodule
```

**What's wrong?**
Because blocking assignments execute sequentially in the order written, `d` is computed using the *previous* value of `x`, rather than the value calculated later in the same block. This produces incorrect combinational behavior.

**Best Practice:** Always assign intermediate variables before they are used in subsequent expressions.

**Corrected order:**
```verilog
always @ (*) begin
  x = a | b;
  d = x & c;
end
```

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_50_36" src="https://github.com/user-attachments/assets/78699e80-4c50-401e-a6f4-73aa80d40fd1" />

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_50_49" src="https://github.com/user-attachments/assets/82179dd2-acbb-4575-aa08-a11ebf002e76" />


---

### Lab 7: Synthesis of the Blocking Caveat Module

Synthesize the corrected version of `blocking_caveat` and review the resulting netlist.

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_50_43" src="https://github.com/user-attachments/assets/6c2cd1ff-130c-4322-99f6-6d7bb657f149" />

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_50_30" src="https://github.com/user-attachments/assets/9c7fe0f5-1ba4-4046-b20f-510a33619af1" />



---

## 5. Summary

- **Gate-Level Simulation (GLS):** Validates netlist functionality, timing, and testability after synthesis.
- **Synthesis-Simulation Mismatch:** Avoided by writing synthesizable, unambiguous RTL code.
- **Blocking vs. Non-Blocking Assignments:** Use blocking (`=`) assignments for combinational logic and non-blocking (`<=`) assignments for sequential logic.
- **Labs:** Reinforce these concepts in practice and highlight common RTL coding pitfalls.

> **Tip:** Always simulate both your RTL and its gate-level netlist, and carefully review any warnings reported by your synthesis and simulation tools.

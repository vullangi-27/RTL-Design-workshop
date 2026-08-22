# Day 3: Combinational and Sequential Optimization

Welcome to Day 3 of this workshop. Today's session focuses on the optimization of combinational and sequential circuits, covering techniques that improve area, timing, and power efficiency in digital designs.

---

## Table of Contents

- [1. Constant Propagation](#1-constant-propagation)
- [2. State Optimization](#2-state-optimization)
- [3. Cloning](#3-cloning)
- [4. Retiming](#4-retiming)
- [5. Labs on Optimization](#5-labs-on-optimization)
  - [Lab 1](#lab-1)
  - [Lab 2](#lab-2)
  - [Lab 3](#lab-3)
  - [Lab 4](#lab-4)
  - [Lab 5](#lab-5)
  - [Lab 6](#lab-6)

---

## 1. Constant Propagation

Constant propagation is an optimization technique applied during synthesis in which variables or signals that always evaluate to a fixed, known value are replaced directly by that value. By eliminating unnecessary variability from the design, the synthesis tool can simplify the resulting logic and produce a more efficient circuit.

**How it works:**
During synthesis, the tool analyzes the design's RTL description to identify signals whose values are constant under all operating conditions. Wherever such a signal is used, it is substituted with its constant value, allowing dependent logic to be simplified or removed entirely.

**Benefits:**
- **Reduced Complexity:** Simplifies logic expressions and shrinks the overall circuit.
- **Performance Improvement:** Removes unnecessary logic from the critical path, reducing propagation delay.
- **Resource Optimization:** Lowers the number of gates and flip-flops required, reducing area and power consumption.

![Constant Propagation Example](https://github.com/user-attachments/assets/d7f06056-66c1-44af-99a8-623fdf5879be)

---

## 2. State Optimization

State optimization improves the efficiency of finite state machines (FSMs) by reducing the number of states, choosing better state encodings, and minimizing the logic required to implement state transitions.

**How it is achieved:**
- **State Reduction:** Functionally equivalent states are identified and merged using state-minimization algorithms, reducing the total state count.
- **State Encoding:** States are assigned encodings (e.g., binary, one-hot, or gray coding) chosen to minimize the resulting combinational logic.
- **Logic Minimization:** Boolean algebra techniques or automated synthesis tools are used to derive compact next-state and output equations.
- **Power Optimization:** Techniques such as clock gating are applied to disable unused portions of the FSM, reducing dynamic power consumption.

---

## 3. Cloning

Cloning is a physical/logical optimization technique in which a logic cell or module is duplicated in order to improve timing, reduce power, or balance the load driven by a single source.

**How it is performed:**
1. Identify critical timing paths using static timing analysis tools.
2. Duplicate the target cell or module that is driving excessive load.
3. Redistribute the fan-out connections between the original and cloned instances to balance the load.
4. Place and route the cloned instance appropriately within the design.
5. Re-verify timing and power improvements through post-optimization analysis.

![Cloning Example](https://github.com/user-attachments/assets/6bdd2c12-02a2-4ea5-895c-98e349b93bac)

---

## 4. Retiming

Retiming is a sequential optimization technique that improves circuit performance by repositioning registers (flip-flops) within the design without altering its functional behavior.

**How it is performed:**
1. **Graph Representation:** The circuit is modeled as a directed graph, with combinational logic as nodes and registers as edge weights.
2. **Register Repositioning:** Registers are moved across combinational logic boundaries to balance path delays between pipeline stages.
3. **Constraint Analysis:** Timing constraints and functional equivalence are preserved throughout the transformation.
4. **Optimization:** Register positions are adjusted to minimize the critical path (and thus the achievable clock period) while optimizing power consumption.

---

## 5. Labs on Optimization

### Lab 1

Verilog code:

```verilog
module opt_check (input a, input b, output y);
    assign y = a ? b : 0;
endmodule
```

**Explanation:**
The statement `assign y = a ? b : 0;` describes the following behavior:
- When `a` is true, `y` takes the value of `b`.
- When `a` is false, `y` is driven to `0`.

Follow the steps described in the [Day 1 Synthesis Lab](https://github.com/Ahtesham18112011/RTL_workshop/tree/main/Day_1#6-synthesis-lab-with-yosys), inserting the following command between `abc -liberty` and `synth -top`:

```shell
opt_clean -purge
```

<img width="1920" height="922" alt="opt1" src="https://github.com/user-attachments/assets/70f7af07-fc0b-43d6-8e4d-32c1668f096d" />
<img width="1920" height="922" alt="opt1gv" src="https://github.com/user-attachments/assets/e7016f23-52fb-484e-b1bf-bb16c4d29710" />

---

### Lab 2

Verilog code:

```verilog
module opt_check2 (input a, input b, output y);
    assign y = a ? 1 : b;
endmodule
```

**Code Analysis:**
This design behaves as a 2-to-1 multiplexer:
- `y = 1` when `a` is true.
- `y = b` when `a` is false.

<img width="1920" height="922" alt="opt2" src="https://github.com/user-attachments/assets/cba982dd-513a-4dba-a673-8ab7dc26e0a9" />
<img width="1920" height="922" alt="opt2gv" src="https://github.com/user-attachments/assets/da9eb498-c3b5-4471-96d5-436c59ee2436" />

---

### Lab 3

Verilog code:

```verilog
module opt_check2 (input a, input b, output y);
    assign y = a ? 1 : b;
endmodule
```

**Functionality:**
As with Lab 2, this design implements a 2-to-1 multiplexer: `y = a ? 1 : b`, producing `1` when `a` is true and `b` otherwise.

<img width="1920" height="922" alt="opt3" src="https://github.com/user-attachments/assets/83ed50ef-7b0b-40b2-99c3-0c22d59fe2ff" />
<img width="1920" height="922" alt="opt3gv" src="https://github.com/user-attachments/assets/49e3abdb-b0db-463c-a6e0-b23604718509" />

---

### Lab 4

Verilog code:

```verilog
module opt_check4 (input a, input b, input c, output y);
    assign y = a ? (b ? (a & c) : c) : (!c);
endmodule
```

**Functionality:**
This module has three inputs (`a`, `b`, `c`) and one output (`y`), implemented using nested ternary (conditional) operators.

Tracing through the logic:
- If `a = 1`, the expression reduces to `y = c` (since `a & c` simplifies to `c` when `a = 1`, regardless of `b`).
- If `a = 0`, the expression evaluates to `y = !c`.

The overall logic therefore simplifies to:

```
y = a ? c : !c
```

<img width="1920" height="922" alt="opt4" src="https://github.com/user-attachments/assets/e58a43ec-27f8-4b63-9a7f-cf0d8a87532e" />
<img width="1920" height="922" alt="opt4gv" src="https://github.com/user-attachments/assets/b0512267-504d-47ab-9be3-77da2cb69f3d" />

---

### Lab 5

Verilog code:

```verilog
module dff_const1 (input clk, input reset, output reg q);
    always @(posedge clk, posedge reset) begin
        if (reset)
            q <= 1'b0;
        else
            q <= 1'b1;
    end
endmodule
```

**Functionality:**
This module implements a D flip-flop with the following behavior:
- Asynchronous reset drives `q` to `0`.
- On each rising clock edge, when not in reset, `q` is loaded with the constant value `1`.

<img width="1920" height="922" alt="dffconst1show" src="https://github.com/user-attachments/assets/bd1362e6-031b-49cb-b1d7-6952266442ce" />
<img width="1920" height="922" alt="gtkwaveconst1" src="https://github.com/user-attachments/assets/344ca9cc-552e-4eec-89a6-0c6ff3f4d835" />

---

### Lab 6

Verilog code:

```verilog
module dff_const2 (input clk, input reset, output reg q);
    always @(posedge clk, posedge reset) begin
        if (reset)
            q <= 1'b1;
        else
            q <= 1'b1;
    end
endmodule
```

**Functionality:**
In this design, `q` is driven to `1` under both the reset and normal operating conditions. As a result, the output remains permanently `1`, regardless of the clock or reset signal — a clear candidate for constant propagation during synthesis.

<img width="1920" height="922" alt="const2show" src="https://github.com/user-attachments/assets/c7780f0d-a699-4a96-bc4a-9801ff58a80b" />
<img width="1920" height="922" alt="const2gtkwave" src="https://github.com/user-attachments/assets/48b25fe0-cb25-4873-96e5-45bc99fa9d43" />

---

## Assignment: D Flip-Flop Constant Optimization

**Objective:** Write and analyze the Verilog code for `dff_const3`, `dff_const4`, and `dff_const5`, then verify their behavior through simulation and synthesis.

**Code Review**

<img width="1920" height="922" alt="const3gvim" src="https://github.com/user-attachments/assets/de262207-ef7f-484b-a77b-c44d9125356f" />
<img width="1920" height="922" alt="const45gvim" src="https://github.com/user-attachments/assets/d5c2418d-bbdb-49ba-af8d-117d71d0f3e9" />

**Synthesis Results**

<img width="1920" height="922" alt="const3show" src="https://github.com/user-attachments/assets/41b95a70-c58b-4c75-9e74-5ce21d44f2a7" />
<img width="1920" height="922" alt="const4show" src="https://github.com/user-attachments/assets/b02cada5-54f0-49a3-bd7c-630e9c9afb91" />
<img width="1920" height="922" alt="const5show" src="https://github.com/user-attachments/assets/9b2bea43-6a83-4a08-97a3-1bd7dae1f3e0" />

**Simulation Results**

<img width="1920" height="922" alt="const4gtkwave" src="https://github.com/user-attachments/assets/b0cb865c-c876-4e03-9c79-30bd1cbb8242" />
<img width="1920" height="922" alt="const3gtkwave" src="https://github.com/user-attachments/assets/f1ebb60b-63fe-4118-ac64-e4aad8b53f9d" />
<img width="1920" height="922" alt="const5gtk" src="https://github.com/user-attachments/assets/93733de7-4e9a-4144-8143-70216d3cd22b" />

---

### Multiple Module Optimization

The following results illustrate optimization applied across multiple interconnected modules.

**Module Views**

<img width="1920" height="922" alt="multoptgv1" src="https://github.com/user-attachments/assets/6bae8a12-5399-4b4c-8ecc-de1cf54991d8" />
<img width="1920" height="922" alt="multoptgv2" src="https://github.com/user-attachments/assets/3e715f0c-adf1-4574-87b0-325d637ce566" />
<img width="1920" height="922" alt="multiple1gv" src="https://github.com/user-attachments/assets/17dffe2e-7db4-4800-bb59-9890e4cc3f76" />

**Synthesis Results**

<img width="1920" height="922" alt="multiopt" src="https://github.com/user-attachments/assets/e71dbc9c-8bf9-4e36-b83a-869f51c4e137" />
<img width="1920" height="922" alt="multipleee1" src="https://github.com/user-attachments/assets/9bab5ca4-5a90-4c5a-8288-1b978b6afedb" />

---

### Counter Optimization

**Code Review**

<img width="1920" height="922" alt="counteroptgvim" src="https://github.com/user-attachments/assets/84f8bd26-2525-4ee0-9141-9aa7b84e29f7" />

**Synthesis Results**

<img width="1920" height="922" alt="countershow" src="https://github.com/user-attachments/assets/cb90f2e6-fc2f-4ef5-9d7f-84ae3cfdfb38" />

<img width="1600" height="647" alt="WhatsApp Image 2026-08-22 at 9 30 16 AM" src="https://github.com/user-attachments/assets/e70d5560-90fd-421d-8fad-af3af59f5d99" />
## Summary

**Focus:** This session covered key optimization techniques for combinational and sequential circuits in digital design, reinforced through hands-on Verilog labs.

**Topics Covered:**
1. **Constant Propagation:** Replacing variables with known constant values to simplify logic and improve circuit efficiency.
2. **State Optimization:** Reducing the number of FSM states and optimizing their encoding to minimize logic and power consumption.
3. **Cloning:** Duplicating logic cells or modules to improve timing closure and balance signal load.
4. **Retiming:** Repositioning registers within a circuit to enhance performance without changing its functional behavior.

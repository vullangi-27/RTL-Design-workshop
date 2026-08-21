# Day 5 — Optimization in Synthesis

Day 5 of the RTL Workshop dives into writing synthesis-friendly Verilog. We look at how `if-else` and `case` constructs behave in combinational logic, why careless coding styles cause the synthesis tool to infer unwanted latches, and how `for` loops and `generate` blocks let us describe scalable hardware without hand-writing repetitive RTL. Twelve hands-on labs reinforce each concept with real synthesis output.

## Table of Contents

1. [If-Else Statements in Verilog](#1-if-else-statements-in-verilog)
2. [Inferred Latches](#2-inferred-latches)
3. [Labs: If-Else and Case Statements](#3-labs-if-else-and-case-statements)
4. [For Loops in Verilog](#4-for-loops-in-verilog)
5. [Generate Blocks in Verilog](#5-generate-blocks-in-verilog)
6. [Ripple Carry Adder (RCA)](#6-ripple-carry-adder-rca)
7. [Labs: Loops and Generate Blocks](#7-labs-loops-and-generate-blocks)
8. [Key Takeaways](#8-key-takeaways)

---

## 1. If-Else Statements in Verilog

`if-else` drives conditional execution inside procedural blocks (`always`, `initial`, tasks, and functions). It's the RTL equivalent of a decision point in software — except every path through it eventually has to map to real hardware.

**Basic syntax**

```verilog
if (condition) begin
    // executes when condition is true
end else begin
    // executes when condition is false
end
```

- `condition` evaluates to true (non-zero) or false (zero).
- `begin ... end` groups multiple statements; it can be dropped for a single statement.
- The `else` branch is optional — but as we'll see below, "optional" doesn't mean "safe to skip."

**Nested if-else**

```verilog
if (condition1) begin
    // condition1 true
end else if (condition2) begin
    // condition2 true
end else begin
    // neither condition true
end
```

## 2. Inferred Latches

A latch gets inferred whenever a combinational (`always @(*)`) block fails to assign a signal on every possible path through the logic. The synthesis tool has no way of knowing you *meant* to hold the previous value, so it builds a latch to preserve it — usually not what you wanted.

**Problem: missing `else` branch**

```verilog
module ex (
    input wire a, b, sel,
    output reg y
);
    always @(a, b, sel) begin
        if (sel == 1'b1)
            y = a;   // no 'else' — y keeps its old value when sel == 0
    end
endmodule
```

Here, when `sel` is `0`, `y` is never driven, so the tool infers a latch to hold its last value.

**Fix: cover every branch explicitly**

```verilog
module ex (
    input wire a, b, sel,
    output reg y
);
    always @(a, b, sel) begin
        case (sel)
            1'b1    : y = a;
            default : y = 1'b0;  // default assignment closes the gap
        endcase
    end
endmodule
```

**Rule of thumb:** in combinational blocks, every output must be assigned on every path — use a final `else` or a `default` case to guarantee it.

## 3. Labs: If-Else and Case Statements

> Setup and simulation/synthesis steps are identical to those covered on Day 1.

### Lab 1 — Incomplete If Statement

```verilog
module incomp_if (input i0, input i1, input i2, output reg y);
    always @(*) begin
        if (i0)
            y <= i1;
    end
endmodule
```
Missing coverage for `i0 == 0` — a latch is inferred on `y`.

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_21_00_16" src="https://github.com/user-attachments/assets/f45da4f7-fff2-45c5-807d-db2bb85c2ae5" />


### Lab 2 — Synthesis Result of Lab 1
Synthesizing Lab 1 confirms the latch: the tool inserts a hold element for `y` instead of pure combinational logic.

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_21_00_46" src="https://github.com/user-attachments/assets/41378693-2589-4af9-8589-ec0531e0a8b9" />

### Lab 3 — Nested If-Else

```verilog
module incomp_if2 (input i0, input i1, input i2, input i3, output reg y);
    always @(*) begin
        if (i0)
            y <= i1;
        else if (i2)
            y <= i3;
    end
endmodule
```
Still incomplete — no branch covers `i0 == 0 && i2 == 0`.

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_21_00_26" src="https://github.com/user-attachments/assets/1ce2a7cc-fb9c-49f6-b22f-13933eff01f0" />

### Lab 4 — Synthesis Result of Lab 3
As expected, synthesis again infers a latch because the last path through the logic leaves `y` unassigned.

### Lab 5 — Complete Case Statement

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_21_00_33" src="https://github.com/user-attachments/assets/7eb00ebf-4ba1-4155-a8a9-bbe47a0d821a" />

```verilog
module comp_case (input i0, input i1, input i2, input [1:0] sel, output reg y);
    always @(*) begin
        case (sel)
            2'b00   : y = i0;
            2'b01   : y = i1;
            default : y = i2;
        endcase
    end
endmodule
```
The `default` branch guarantees full coverage — no latch here.

### Lab 6 — Synthesis Result of Lab 5
The synthesized netlist is purely combinational, exactly as intended, confirming the `default` case did its job.

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_59_43" src="https://github.com/user-attachments/assets/b7e0244b-bfb8-40db-86e0-d98bf5cf9e6f" />

### Lab 7 — Incomplete Case Handling

```verilog
module bad_case (
    input i0, input i1, input i2, input i3,
    input [1:0] sel,
    output reg y
);
    always @(*) begin
        case (sel)
            2'b00 : y = i0;
            2'b01 : y = i1;
            2'b10 : y = i2;
            2'b1?: y = i3;   // '?' is a wildcard — handle it with care
        endcase
    end
endmodule


```

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_59_20" src="https://github.com/user-attachments/assets/62c0644b-c6e3-43b3-84c8-801bc80ffa01" />

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_58_55" src="https://github.com/user-attachments/assets/25f2c334-1668-4983-977b-65e740dbbbec" />

Wildcard patterns can quietly overlap or leave gaps in coverage; they deserve extra scrutiny during review.

### Lab 8 — Partial Assignments in Case

```verilog
module partial_case_assign (
    input i0, input i1, input i2,
    input [1:0] sel,
    output reg y, output reg x
);
    always @(*) begin
        case (sel)
            2'b00: begin
                y = i0;
                x = i2;
            end
            2'b01: y = i1;
            default: begin
                x = i1;
                y = i2;
            end
        endcase
    end
endmodule
```

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_21_02_06" src="https://github.com/user-attachments/assets/2a5ee5c3-d2e0-4dde-aeac-dbe567d0cea8" />

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_21_01_54" src="https://github.com/user-attachments/assets/61f0162e-c989-4b11-9581-bb3b3e815ab9" />

Notice that `x` isn't assigned in the `2'b01` branch — a partial assignment like this is just as capable of inferring a latch as a missing `else`, since each *output signal* needs coverage on every path.

## 4. For Loops in Verilog

A `for` loop repeats statements inside a procedural block based on a loop counter — handy for describing regular, repetitive logic concisely.

**Syntax**

```verilog
for (initialization; condition; increment) begin
    // statements
end
```

- Must live inside a procedural block (`initial`, `always`, task, or function).
- Synthesizable only when the number of iterations is fixed at compile time — no loops with runtime-dependent bounds.

**Example: 4-to-1 MUX using a for loop**

```verilog
module mux_4to1_for_loop (
    input wire [3:0] data,  // 4 input lines
    input wire [1:0] sel,   // 2-bit select
    output reg y            // output
);
    integer i;
    always @(data, sel) begin
        y = 1'b0;  // default output
        for (i = 0; i < 4; i = i + 1) begin
            if (i == sel)
                y = data[i];
        end
    end
endmodule
```


## 5. Generate Blocks in Verilog

A `generate` block builds hardware structures — module instances, repeated logic — at elaboration time. It's typically paired with a `for` loop and a `genvar`.

```verilog
genvar i;
generate
    for (i = 0; i < 4; i = i + 1) begin : gen_loop
        and_gate and_inst (.a(in[i]), .b(in[i+1]), .y(out[i]));
    end
endgenerate
```
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_21_01_32" src="https://github.com/user-attachments/assets/57f971fa-ea93-4808-848c-3acaa6959c58" />

Where a procedural `for` loop describes repeated *behavior*, a `generate` loop describes repeated *structure* — multiple instances of hardware wired up automatically.

## 6. Ripple Carry Adder (RCA)

A Ripple Carry Adder sums two binary numbers using a chain of full adders: adding `n` bits takes `n` full adders, with each stage's carry-out feeding the next stage's carry-in. It's a natural fit for a `generate` loop, since the same full-adder cell is instantiated repeatedly with different bit indices.


### Lab 9 — 4-to-1 MUX Using a For Loop

```verilog
module mux_generate (
    input i0, input i1, input i2, input i3,
    input [1:0] sel,
    output reg y
);
    wire [3:0] i_int;
    assign i_int = {i3, i2, i1, i0};
    integer k;
    always @(*) begin
        for (k = 0; k < 4; k = k + 1) begin
            if (k == sel)
                y = i_int[k];
        end
    end
endmodule
```
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_21_01_24" src="https://github.com/user-attachments/assets/6a87b6f7-fa60-4ebf-a561-8a055dd57836" />

<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_21_01_41" src="https://github.com/user-attachments/assets/15ee65c6-48a2-4f3e-9908-c33a75e31a4b" />
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_21_01_32" src="https://github.com/user-attachments/assets/57f971fa-ea93-4808-848c-3acaa6959c58" />
### Lab 10 — 8-to-1 Demux Using Case

```verilog
module demux_case (
    output o0, output o1, output o2, output o3,
    output o4, output o5, output o6, output o7,
    input [2:0] sel,
    input i
);
    reg [7:0] y_int;
    assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int;
    always @(*) begin
        y_int = 8'b0;
        case (sel)
            3'b000 : y_int[0] = i;
            3'b001 : y_int[1] = i;
            3'b010 : y_int[2] = i;
            3'b011 : y_int[3] = i;
            3'b100 : y_int[4] = i;
            3'b101 : y_int[5] = i;
            3'b110 : y_int[6] = i;
            3'b111 : y_int[7] = i;
        endcase
    end
endmodule
```
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_56_42" src="https://github.com/user-attachments/assets/1324abe9-e1d3-4539-90c4-6c5be59f800c" />
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_57_00" src="https://github.com/user-attachments/assets/d0a10f4f-1b8e-4869-bf88-55bfbc22d795" />
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_56_50" src="https://github.com/user-attachments/assets/85132b7a-c4a9-4515-bff1-2c885f03e9e2" />


### Lab 11 — 8-to-1 Demux Using a For Loop

```verilog
module demux_generate (
    output o0, output o1, output o2, output o3,
    output o4, output o5, output o6, output o7,
    input [2:0] sel,
    input i
);
    reg [7:0] y_int;
    assign {o7, o6, o5, o4, o3, o2, o1, o0} = y_int;
    integer k;
    always @(*) begin
        y_int = 8'b0;
        for (k = 0; k < 8; k = k + 1) begin
            if (k == sel)
                y_int[k] = i;
        end
    end
endmodule
```
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_57_17" src="https://github.com/user-attachments/assets/63401572-4896-4db0-a64a-35f6be2a4299" />
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_57_08" src="https://github.com/user-attachments/assets/a1f92b24-8983-40f7-98e5-d5958b0468e2" />

Compare Labs 10 and 11: the `for`-loop version replaces eight near-identical `case` lines with four lines of logic — the same behavior, far less repetition.

### Lab 12 — 8-bit Ripple Carry Adder with a Generate Block

```verilog
module rca (
    input  [7:0] num1,
    input  [7:0] num2,
    output [8:0] sum
);
    wire [7:0] int_sum;
    wire [7:0] int_co;

    genvar i;
    generate
        for (i = 1; i < 8; i = i + 1) begin
            fa u_fa_1 (.a(num1[i]), .b(num2[i]), .c(int_co[i-1]), .co(int_co[i]), .sum(int_sum[i]));
        end
    endgenerate

    fa u_fa_0 (.a(num1[0]), .b(num2[0]), .c(1'b0), .co(int_co[0]), .sum(int_sum[0]));

    assign sum[7:0] = int_sum;
    assign sum[8]   = int_co[7];
endmodule
```

**Full adder submodule**

```verilog
module fa (input a, input b, input c, output co, output sum);
    assign {co, sum} = a + b + c;
endmodule
```
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_58_14" src="https://github.com/user-attachments/assets/a0506dab-7589-4890-a315-4c52722ff2d9" />
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_58_36" src="https://github.com/user-attachments/assets/019adbe3-be7c-481c-baa1-775ab346fc4f" />
<img width="1280" height="800" alt="VirtualBox_vsdworkshop_21_08_2026_20_58_29" src="https://github.com/user-attachments/assets/bc4020cb-f1ce-4990-96fc-500327e32845" />



The generate loop wires up bits 1–7 automatically, while bit 0 (with its carry-in tied to `1'b0`) is instantiated once outside the loop — a clean way to handle the one stage that's structurally different from the rest.

## 8. Key Takeaways

- Write complete `if-else` and `case` statements — every combinational output needs a value on every possible path, or the tool will infer a latch you didn't ask for.
- A `default` case or trailing `else` is the simplest guarantee of full coverage.
- Watch for **partial** assignments too: a signal missing from just one branch is enough to trigger a latch.
- `for` loops and `generate` blocks turn repetitive RTL into compact, scalable, synthesizable code.
- Always verify intent against the synthesized netlist, not just the simulation — that's where latch inference actually shows up.

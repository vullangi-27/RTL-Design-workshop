# Design and Physical Implementation of a Smart Parking Management Controller using SKY130 and OpenLane

![Technology](https://img.shields.io/badge/Technology-SKY130%20HD-blue)
![Design](https://img.shields.io/badge/Design-RTL%20to%20GDSII-green)
![Simulation](https://img.shields.io/badge/RTL%20%2B%20GLS-10%2F10%20Tests%20Pass-brightgreen)
![DRC](https://img.shields.io/badge/DRC-0%20Violations-brightgreen)
![LVS](https://img.shields.io/badge/LVS-0%20Errors-brightgreen)
![STA](https://img.shields.io/badge/STA-16%20PVT%20Corners-orange)

## 1. Project Overview

Parking facilities need a reliable way to count vehicles, control entry and exit gates, and tell drivers when no space is available. A small hardware controller can do this deterministically, without a processor or software.

This project presents the **RTL design, simulation, synthesis, gate-level simulation, physical implementation, and multi-corner timing verification of a Smart Parking Management Controller** using Verilog HDL and an open-source ASIC flow.

The controller tracks the number of occupied slots in an 8-slot parking area. A finite-state machine (FSM) handles vehicle entry and exit requests, drives the entry and exit gate signals, and raises a parking-full flag when every slot is taken.

The design is implemented with the **SkyWater SKY130 HD standard-cell library** through an OpenLane RTL-to-GDSII flow, and is verified by RTL simulation, gate-level simulation (GLS), Magic DRC, Netgen LVS, and Static Timing Analysis (STA) across 16 PVT corners.

## 2. Problem Statement

Manual or loosely controlled parking entry can cause overcrowding, wrong slot counts, and gates opening when the facility is full. This project addresses the design of a sequential digital controller that:

- Counts vehicles entering and leaving a parking area.
- Opens the entry or exit gate for one controlled handshake.
- Refuses entry when all slots are occupied.
- Ignores an exit request when the parking area is empty.
- Reports occupied slots, available slots, and a full indication.
- Can be synthesized, placed, routed, and verified as an ASIC design.

## 3. Project Objectives

1. Develop a synthesizable Verilog RTL module for parking control.
2. Implement a four-state FSM with gate and full-state handling.
3. Verify the design with a self-checking testbench covering normal and corner cases.
4. Synthesize the RTL into SKY130 HD standard cells.
5. Verify the synthesized netlist with gate-level simulation and compare it with RTL results.
6. Complete floorplanning, power distribution, placement, clock-tree synthesis, routing, and GDSII generation.
7. Evaluate setup and hold timing across 16 PVT corners.
8. Review final physical verification: DRC, LVS, and antenna checks.

## 4. System Architecture

The following block diagram shows the controller's inputs, FSM, occupancy counter, and outputs.

<img width="1920" height="923" alt="block_diagram" src="https://github.com/user-attachments/assets/41109124-b1f9-4d49-ba16-2f65f62a37b8" />


1. **Input Signals:** The controller receives the clock, reset, a vehicle-entry request, and a vehicle-exit request.
2. **Finite State Machine:** The FSM moves between IDLE, ENTRY, EXIT, and FULL states. Exit has priority over entry.
3. **Occupancy Counter:** A 4-bit counter tracks occupied slots from 0 to 8.
4. **Output Logic:** The controller drives the gate signals, the occupied and available slot counts, and the parking-full flag.

### 4.1 Input Signals

| Signal | Description |
|---|---|
| `clk` | Clock input for sequential state updates |
| `reset` | Synchronous, active-high reset. Returns the FSM to IDLE, occupied slots to 0, available slots to 8, and clears the gates and full flag |
| `vehicle_entry` | One-cycle pulse from the entry sensor: a car wants to enter |
| `vehicle_exit` | One-cycle pulse from the exit sensor: a car wants to leave |

### 4.2 Output Signals

| Signal | Description |
|---|---|
| `entry_gate` | Asserted while the entry gate is open (ENTRY state) |
| `exit_gate` | Asserted while the exit gate is open (EXIT state) |
| `occupied_slots[3:0]` | Number of occupied slots, 0 to 8 |
| `available_slots[3:0]` | Number of free slots, 8 minus occupied |
| `parking_full` | Asserted when all 8 slots are occupied |

## 5. Finite-State Machine (FSM)

The RTL uses a four-state FSM with two-bit state encoding.

| State | RTL encoding | Function |
|---|---|---|
| IDLE | `2'b00` | Waiting for a request, gates closed |
| ENTRY | `2'b01` | Entry gate open for an accepted vehicle |
| EXIT | `2'b10` | Exit gate open for a leaving vehicle |
| FULL | `2'b11` | All slots occupied, entry refused |

<img width="1920" height="923" alt="fsm_diagram" src="https://github.com/user-attachments/assets/2f00ad33-cc2f-4a66-befc-545b539e64a2" />

### 5.1 State Operation (observed in simulation)

**IDLE**
- With free slots, a `vehicle_entry` request moves the FSM to ENTRY and opens the entry gate.
- A `vehicle_exit` request with at least one occupied slot moves the FSM to EXIT and opens the exit gate.
- A `vehicle_exit` request when the parking area is empty is ignored.
- If entry and exit are requested together, the exit has priority.
- If an entry is requested while the lot is already full, the FSM goes to FULL as a safety measure.

**ENTRY**
- The entry gate is open. Occupancy increases by one on the next clock, and the FSM returns to IDLE.
- When the last free slot is taken, the FSM goes to FULL instead of IDLE.

**EXIT**
- The exit gate is open. Occupancy decreases by one and the FSM returns to IDLE on the next clock.

**FULL**
- `parking_full` is asserted and `available_slots` is 0.
- Entry requests are ignored.
- An exit request moves the FSM to EXIT, and after it completes the FSM returns to IDLE with `parking_full` cleared.

## 6. Occupancy Counting Logic

The design supports 8 slots (`TOTAL_SLOTS = 8`). Two four-bit registers, `occupied_slots` and `available_slots`, are updated together: an entry adds one to occupied and subtracts one from available, and an exit does the opposite. Both updates are guarded so the counts stay between 0 and 8, and the full flag is set from the next occupancy value:

```text
occupied + available = 8   (always)
parking_full <= (occ_next == 8)
```

The testbench checks that the counter never exceeds 8 and never goes below 0, including the cases of entry while full and exit while empty.

## 7. Hardware and Software Tools

| Tool / Technology | Purpose |
|---|---|
| Verilog HDL | RTL description |
| Icarus Verilog 10.1 | RTL and gate-level simulation |
| GTKWave 3.3.86 | Waveform inspection |
| Yosys | Logic synthesis and synthesis statistics |
| OpenLane v0.21 | Automated RTL-to-GDSII flow |
| OpenROAD / TritonCTS / TritonRoute | Placement, clock-tree synthesis, and routing |
| SKY130 HD PDK (`sky130A`) | Standard-cell library and design rules |
| OpenSTA | Static timing analysis |
| Magic | Layout inspection, DRC, and GDS generation |
| Netgen | Layout-versus-schematic comparison |
| Docker | Running the OpenLane container |
| Git and GitHub | Version control and documentation |

## 8. Complete RTL-to-GDSII Implementation Flow

```text
Verilog RTL
    |
    v
RTL Simulation
    |
    v
Logic Synthesis
    |
    v
Gate-Level Simulation
    |
    v
Floorplanning and Power Distribution
    |
    v
Placement
    |
    v
Clock-Tree Synthesis
    |
    v
Global and Detailed Routing
    |
    v
Parasitic Extraction and Timing Analysis
    |
    v
Physical Verification (DRC, LVS, Antenna)
    |
    v
GDSII Layout
```

## 8.1 RTL Design

The controller is described in Verilog HDL using:
- A two-bit FSM state register.
- A four-bit occupancy counter.
- Sequential state-transition and counter logic.
- Combinational next-state and next-count logic, with registered outputs: the gates, slot counts, and full flag all come directly from flip-flops.

**RTL source:** `src/smart_parking_controller.v`

## 8.2 RTL Simulation

The self-checking testbench applies a reset and then ten scenarios: reset, one vehicle entry, multiple entries (4 cars), vehicle exit, filling all slots, entry when full, exit from full, exit when empty, simultaneous entry and exit, and a final reset. Each scenario prints PASS or FAIL and a final summary. The testbench also checks that occupied plus available slots always equals 8, captures the gate outputs during each operation, and includes a watchdog timeout.

### RTL Code

Source file: `src/smart_parking_controller.v`

```verilog
module smart_parking_controller (
    input            clk,
    input            reset,            // synchronous, active high
    input            vehicle_entry,    // 1-cycle pulse: car wants to enter
    input            vehicle_exit,     // 1-cycle pulse: car wants to leave
    output reg       entry_gate,       // 1 while entry barrier is open
    output reg       exit_gate,        // 1 while exit  barrier is open
    output reg       parking_full,     // 1 when occupied_slots == 8
    output reg [3:0] available_slots,  // 0..8
    output reg [3:0] occupied_slots    // 0..8
);

    //-------------------------------------------------------------------------
    // Constants
    //-------------------------------------------------------------------------
    localparam [3:0] TOTAL_SLOTS = 4'd8;

    // FSM state encoding (2 bits are enough for 4 states)
    localparam [1:0] IDLE  = 2'd0;
    localparam [1:0] ENTRY = 2'd1;
    localparam [1:0] EXIT  = 2'd2;
    localparam [1:0] FULL  = 2'd3;

    //-------------------------------------------------------------------------
    // Internal signals
    //-------------------------------------------------------------------------
    reg [1:0] state;          // current state (flip-flops)
    reg [1:0] next_state;     // next state    (combinational)
    reg [3:0] occ_next;       // next value of occupied_slots  (combinational)
    reg [3:0] avail_next;     // next value of available_slots (combinational)

    //-------------------------------------------------------------------------
    // 1) State register (sequential)
    //-------------------------------------------------------------------------
    always @(posedge clk) begin
        if (reset)
            state <= IDLE;
        else
            state <= next_state;
    end

    //-------------------------------------------------------------------------
    // 2) Next-state logic (combinational)
    //    Default assignment at the top prevents latches.
    //-------------------------------------------------------------------------
    always @(*) begin
        next_state = state;
        case (state)
            IDLE: begin
                if (vehicle_exit) begin                 // exit has priority
                    if (occupied_slots != 4'd0)
                        next_state = EXIT;              // someone is inside
                    // else: lot empty -> reject, stay in IDLE
                end
                else if (vehicle_entry) begin
                    if (occupied_slots != TOTAL_SLOTS)
                        next_state = ENTRY;             // free slot exists
                    else
                        next_state = FULL;              // safety: lot is full
                end
            end

            ENTRY: begin
                // This vehicle takes the last slot -> go to FULL
                if (occupied_slots == (TOTAL_SLOTS - 4'd1))
                    next_state = FULL;
                else
                    next_state = IDLE;
            end

            EXIT: begin
                next_state = IDLE;
            end

            FULL: begin
                // vehicle_entry is ignored here; only an exit can free a slot
                if (vehicle_exit)
                    next_state = EXIT;
            end

            default: next_state = IDLE;                 // recover from illegal state
        endcase
    end

    //-------------------------------------------------------------------------
    // 3) Counter next-value logic (combinational)
    //    Bounds are guarded so the counters can never go below 0 or above 8.
    //-------------------------------------------------------------------------
    always @(*) begin
        occ_next   = occupied_slots;
        avail_next = available_slots;
        if (state == ENTRY && occupied_slots < TOTAL_SLOTS) begin
            occ_next   = occupied_slots   + 4'd1;
            avail_next = available_slots  - 4'd1;
        end
        else if (state == EXIT && occupied_slots != 4'd0) begin
            occ_next   = occupied_slots   - 4'd1;
            avail_next = available_slots  + 4'd1;
        end
    end

    //-------------------------------------------------------------------------
    // 4) Output / counter registers (sequential)
    //    All outputs come directly from flip-flops (glitch-free, STA friendly).
    //-------------------------------------------------------------------------
    always @(posedge clk) begin
        if (reset) begin
            occupied_slots  <= 4'd0;
            available_slots <= TOTAL_SLOTS;   // 8
            parking_full    <= 1'b0;
            entry_gate      <= 1'b0;
            exit_gate       <= 1'b0;
        end
        else begin
            occupied_slots  <= occ_next;
            available_slots <= avail_next;
            parking_full    <= (occ_next == TOTAL_SLOTS);
            entry_gate      <= (next_state == ENTRY);  // high while state==ENTRY
            exit_gate       <= (next_state == EXIT);   // high while state==EXIT
        end
    end
endmodule
```

### Testbench Code

Source file: `src/tb_smart_parking_controller.v`

```verilog
//=============================================================================
// File   : tb_smart_parking_controller.v
// Purpose: Self-checking testbench for smart_parking_controller.
//          Vehicle sensors are simulated purely by driving vehicle_entry /
//          vehicle_exit from this testbench. (Simulation only - not synthesized)
//=============================================================================
`timescale 1ns/1ps

module tb_smart_parking_controller;

    // DUT connections
    reg        clk;
    reg        reset;
    reg        vehicle_entry;
    reg        vehicle_exit;
    wire       entry_gate;
    wire       exit_gate;
    wire       parking_full;
    wire [3:0] available_slots;
    wire [3:0] occupied_slots;

    // Bookkeeping
    reg     entry_seen;      // entry_gate value captured DURING the operation
    reg     exit_seen;       // exit_gate  value captured DURING the operation
    reg     test_ok;
    integer pass_count;
    integer fail_count;
    integer i;

    // Device under test
    smart_parking_controller dut (
        .clk             (clk),
        .reset           (reset),
        .vehicle_entry   (vehicle_entry),
        .vehicle_exit    (vehicle_exit),
        .entry_gate      (entry_gate),
        .exit_gate       (exit_gate),
        .parking_full    (parking_full),
        .available_slots (available_slots),
        .occupied_slots  (occupied_slots)
    );

    // 10 ns clock
    always #5 clk = ~clk;

    // Print important signals whenever any of them changes
    initial begin
        $monitor("t=%0t | state=%0d | entry=%b exit=%b | entry_gate=%b exit_gate=%b | occ=%0d avail=%0d full=%b",
                 $time, dut.state, vehicle_entry, vehicle_exit,
                 entry_gate, exit_gate, occupied_slots, available_slots, parking_full);
    end

    //-------------------------------------------------------------------------
    // Helper tasks
    //-------------------------------------------------------------------------

    // Apply a synchronous reset for 3 clock cycles
    task apply_reset;
        begin
            @(posedge clk); #1;
            reset = 1'b1; vehicle_entry = 1'b0; vehicle_exit = 1'b0;
            repeat (3) @(posedge clk);
            #1 reset = 1'b0;
            @(posedge clk); #1;
        end
    endtask

    // Drive a ONE-cycle request, capture gate values while the FSM is in
    // ENTRY/EXIT, then wait until the operation is finished + idle gap.
    task pulse;
        input e;
        input x;
        begin
            @(posedge clk); #1;
            vehicle_entry = e; vehicle_exit = x;
            @(posedge clk); #1;               // DUT samples request on this edge
            vehicle_entry = 1'b0; vehicle_exit = 1'b0;
            entry_seen = entry_gate;          // gates are high NOW if accepted
            exit_seen  = exit_gate;
            @(posedge clk); #1;               // counters update on this edge
            repeat (2) @(posedge clk);        // idle gap for waveform viewing
            #1;
        end
    endtask

    task vehicle_in;   begin pulse(1'b1, 1'b0); end endtask
    task vehicle_out;  begin pulse(1'b0, 1'b1); end endtask
    task both;         begin pulse(1'b1, 1'b1); end endtask

    // Check counters + full flag (also checks occupied + available == 8)
    task expect_state;
        input [3:0] o;
        input [3:0] a;
        input       f;
        begin
            if (occupied_slots !== o || available_slots !== a || parking_full !== f
                || (occupied_slots + available_slots) !== 4'd8) begin
                test_ok = 1'b0;
                $display("   MISMATCH: occ=%0d(exp %0d) avail=%0d(exp %0d) full=%b(exp %b)",
                         occupied_slots, o, available_slots, a, parking_full, f);
            end
        end
    endtask

    // Check live gate outputs
    task expect_gates;
        input eg;
        input xg;
        begin
            if (entry_gate !== eg || exit_gate !== xg) begin
                test_ok = 1'b0;
                $display("   MISMATCH: entry_gate=%b(exp %b) exit_gate=%b(exp %b)",
                         entry_gate, eg, exit_gate, xg);
            end
        end
    endtask

    // Check gate values captured during the operation
    task expect_seen;
        input eg;
        input xg;
        begin
            if (entry_seen !== eg || exit_seen !== xg) begin
                test_ok = 1'b0;
                $display("   MISMATCH: gate seen entry=%b(exp %b) exit=%b(exp %b)",
                         entry_seen, eg, exit_seen, xg);
            end
        end
    endtask

    // Print PASS/FAIL for one test and clear the flag
    task report;
        input [8*40-1:0] name;
        begin
            if (test_ok) begin
                $display(">>> %0s : PASS", name);
                pass_count = pass_count + 1;
            end else begin
                $display(">>> %0s : FAIL", name);
                fail_count = fail_count + 1;
            end
            test_ok = 1'b1;
        end
    endtask

    //-------------------------------------------------------------------------
    // Main stimulus
    //-------------------------------------------------------------------------
    initial begin
        $dumpfile("smart_parking.vcd");
        $dumpvars(0, tb_smart_parking_controller);

        clk = 1'b0; reset = 1'b1; vehicle_entry = 1'b0; vehicle_exit = 1'b0;
        entry_seen = 1'b0; exit_seen = 1'b0;
        test_ok = 1'b1; pass_count = 0; fail_count = 0;

        // TEST 1 - RESET
        $display("\n--- TEST 1: RESET ---");
        apply_reset;
        expect_state(4'd0, 4'd8, 1'b0);
        expect_gates(1'b0, 1'b0);
        report("TEST 1  RESET");

        // TEST 2 - ONE VEHICLE ENTRY
        $display("\n--- TEST 2: ONE VEHICLE ENTRY ---");
        vehicle_in;
        expect_seen(1'b1, 1'b0);              // entry_gate asserted during entry
        expect_state(4'd1, 4'd7, 1'b0);
        report("TEST 2  ONE VEHICLE ENTRY");

        // TEST 3 - MULTIPLE ENTRIES (total 4 vehicles)
        $display("\n--- TEST 3: MULTIPLE ENTRIES ---");
        for (i = 0; i < 3; i = i + 1) begin
            vehicle_in;
            if (entry_seen !== 1'b1) test_ok = 1'b0;
        end
        expect_state(4'd4, 4'd4, 1'b0);
        report("TEST 3  MULTIPLE ENTRIES (4 cars)");

        // TEST 4 - VEHICLE EXIT
        $display("\n--- TEST 4: VEHICLE EXIT ---");
        vehicle_out;
        expect_seen(1'b0, 1'b1);              // exit_gate asserted during exit
        expect_state(4'd3, 4'd5, 1'b0);
        report("TEST 4  VEHICLE EXIT");

        // TEST 5 - FILL ALL SLOTS (3 -> 8)
        $display("\n--- TEST 5: FILL ALL PARKING SLOTS ---");
        for (i = 0; i < 5; i = i + 1) vehicle_in;
        expect_state(4'd8, 4'd0, 1'b1);
        report("TEST 5  FILL ALL SLOTS");

        // TEST 6 - ENTRY WHEN FULL
        $display("\n--- TEST 6: ENTRY WHEN PARKING IS FULL ---");
        vehicle_in;
        expect_seen(1'b0, 1'b0);              // gate must NOT open
        expect_gates(1'b0, 1'b0);
        expect_state(4'd8, 4'd0, 1'b1);
        report("TEST 6  ENTRY WHEN FULL");

        // TEST 7 - EXIT FROM FULL
        $display("\n--- TEST 7: EXIT FROM FULL PARKING ---");
        vehicle_out;
        expect_seen(1'b0, 1'b1);
        expect_state(4'd7, 4'd1, 1'b0);
        report("TEST 7  EXIT FROM FULL");

        // TEST 8 - EXIT WHEN EMPTY
        $display("\n--- TEST 8: EXIT WHEN PARKING IS EMPTY ---");
        apply_reset;
        vehicle_out;
        expect_seen(1'b0, 1'b0);              // no gate for a non-existent car
        expect_state(4'd0, 4'd8, 1'b0);
        report("TEST 8  EXIT WHEN EMPTY");

        // TEST 9 - SIMULTANEOUS ENTRY AND EXIT (exit has priority)
        $display("\n--- TEST 9: SIMULTANEOUS ENTRY AND EXIT ---");
        for (i = 0; i < 3; i = i + 1) vehicle_in;     // park 3 cars
        expect_state(4'd3, 4'd5, 1'b0);
        both;                                          // entry=1 and exit=1
        expect_seen(1'b0, 1'b1);              // only exit_gate opens
        expect_state(4'd2, 4'd6, 1'b0);       // net result: one car left
        report("TEST 9  SIMULTANEOUS ENTRY+EXIT");

        // TEST 10 - FINAL RESET (applied while cars are parked)
        $display("\n--- TEST 10: FINAL RESET ---");
        apply_reset;
        expect_state(4'd0, 4'd8, 1'b0);
        expect_gates(1'b0, 1'b0);
        report("TEST 10 FINAL RESET");

        $display("\n=========================================");
        $display(" SUMMARY: %0d PASSED, %0d FAILED", pass_count, fail_count);
        $display("=========================================\n");
        #50;
        $finish;
    end

    // Watchdog: stop the simulation if something hangs
    initial begin
        #100000;
        $display("ERROR: simulation timeout");
        $finish;
    end

endmodule
```

### Simulation

Commands, run from the `src` directory:

```bash
iverilog -o rtl_sim smart_parking_controller.v tb_smart_parking_controller.v
vvp rtl_sim | tee rtl_sim.log
gtkwave smart_parking.vcd
```

**Result: 10 PASSED, 0 FAILED**

<img width="1920" height="923" alt="c07_1" src="https://github.com/user-attachments/assets/3b58c599-5457-4a30-a8fc-294cdcf154c6" />


### RTL Waveforms

<img width="1920" height="923" alt="c07_2" src="https://github.com/user-attachments/assets/d380c1ce-220e-4005-ba15-9acd0166db10" />

## 8.3 Logic Synthesis

Yosys, run through OpenLane, converts the RTL into a gate-level netlist mapped to SKY130 HD standard cells.

```tcl
./flow.tcl -interactive
package require openlane 0.9
prep -design smart_parking -tag run1 -overwrite
run_synthesis
```

Synthesis result:

| Metric | Value |
|---|---|
| Total cells | 117 |
| Flip-flops (`dfxtp_2`) | 17 |
| Chip area | 1078.53 µm² |
| Synthesis WNS / TNS | 0.00 ns / 0.00 ns |

Cell usage:

| Cell | Count |
|---|---|
| `inv_2` | 28 |
| `dfxtp_2` | 17 |
| `mux2_1` | 12 |
| `nor2_2` | 10 |
| `nand2_2`, `o21a_2`, `or2_2` | 5 each |
| `buf_1`, `o21ai_2`, `o22a_2` | 4 each |
| `and2_2`, `nor3_2`, `o221a_2` | 3 each |
| Other (10 cell types) | 1 to 2 each |

Yosys re-encoded the FSM as one-hot, so the netlist has four state flip-flops (`state[0]` to `state[3]`) instead of a 2-bit register.

<img width="1920" height="923" alt="4" src="https://github.com/user-attachments/assets/374979b2-f4f3-4077-b5b6-020e6f9e6a77" />

### Synthesis Report

<img width="1920" height="923" alt="10" src="https://github.com/user-attachments/assets/aef4864a-be3f-4407-b8ad-6069071fd6a1" />

### Synthesized Gate-Level Schematic
<img width="1920" height="923" alt="schematic" src="https://github.com/user-attachments/assets/840d414d-90c3-4e22-8524-79a70affa373" />


## 8.4 Gate-Level Simulation (GLS)

The synthesized netlist is simulated with the same ten scenarios as the RTL testbench, using the SKY130 HD cell models.

```bash
cd gls
iverilog -o gls_sim tb_gls.v smart_parking_controller.synthesis.v \
  $PDK_ROOT/sky130A/libs.ref/sky130_fd_sc_hd/verilog/primitives.v used_models.v
vvp gls_sim | tee gls_sim.log
gtkwave smart_parking_gls.vcd
```

**Result: 10 PASSED, 0 FAILED**

<img width="1920" height="923" alt="gls_5" src="https://github.com/user-attachments/assets/6461d8cd-bd61-429d-9733-c3542d02fd81" />


### RTL versus GLS Comparison

The monitored output lines (gates, occupied slots, available slots, full flag, and inputs) are identical between RTL and GLS: 68 lines each, and `diff` returns no differences.

```bash
diff <(grep "^t=" src/rtl_sim.log | sed 's/state=[^|]*|//') \
     <(grep "^t=" gls/gls_sim.log | sed 's/state_onehot=[^|]*|//')
```

The FSM state is not compared directly, because synthesis changed its encoding:

| RTL state | RTL encoding | GLS one-hot |
|---|---|---|
| IDLE | 0 | `0001` |
| ENTRY | 1 | `0100` |
| EXIT | 2 | `0010` |
| FULL | 3 | `1000` |


### GLS Waveforms
<img width="1920" height="923" alt="gls_7" src="https://github.com/user-attachments/assets/d2e6d748-fd91-4976-a041-b6e83fa4c26e" />

- Icarus Verilog 10.1 cannot parse the complete `sky130_fd_sc_hd.v` model file.
- The workaround is to preprocess the models (`iverilog -E -DFUNCTIONAL -DUNIT_DELAY=#0`) and extract only the modules used by the netlist, including their base modules. The result is `used_models.v`.
- The GLS testbench prints the one-hot state through an intermediate wire, because `$monitor` cannot take concatenated hierarchical escaped names directly.

## 8.5 Floorplanning

Floorplanning defines the core and die area, placement rows, and power distribution network (PDN) for the design.

- `FP_CORE_UTIL = 20`, a sparse core utilization suited to a small design.
- The PDN is generated inside the floorplan stage. In interactive mode, `gen_pdn` must **not** be run again after `run_floorplan`, otherwise OpenROAD fails with `invalid command name "NULL"`.

### Floorplan
<img width="1920" height="923" alt="floorplannig_10" src="https://github.com/user-attachments/assets/d778d6c6-7673-45d6-aaa6-d40befe2d7ea" />
<img width="1920" height="923" alt="floorplanning_15" src="https://github.com/user-attachments/assets/5126ff93-e2b6-4432-9162-46bb5fb6df97" />


## 8.6 Placement

Placement assigns physical locations to the standard cells inside the floorplan, with `PL_TARGET_DENSITY = 0.3`.

<img width="1920" height="923" alt="placement_11" src="https://github.com/user-attachments/assets/66c050bc-96e7-42cf-a9d0-636cb56bc5f5" />
<img width="1920" height="923" alt="placement_16" src="https://github.com/user-attachments/assets/24486199-c320-441d-ba00-52c4b153d511" />

## 8.7 Clock-Tree Synthesis (CTS)

TritonCTS builds the clock distribution network. In the final run, the `clk` net has 17 sinks (the 17 flip-flops).
<img width="1920" height="923" alt="cts_13" src="https://github.com/user-attachments/assets/4a3bb1c0-4c7b-43e2-b7be-19d82896a4a6" />
<img width="1920" height="923" alt="cts_16" src="https://github.com/user-attachments/assets/f66486ac-4314-47d1-b5c0-381111912b6f" />


## 8.8 Routing

Global and detailed routing create the metal interconnects between cells and ports.

| Metric | Value |
|---|---|
| TritonRoute violations | 0 |
| Antenna violations | 0 |
| Wire length | 2744 µm |
| Vias | 839 |

<img width="1920" height="923" alt="routing_14" src="https://github.com/user-attachments/assets/41137cc2-9364-426a-99f2-c2fd86d7b5ee" />
<img width="1920" height="923" alt="routing_17" src="https://github.com/user-attachments/assets/a41da141-6d23-4039-9525-b88f4a217c6f" />

## 8.9 Complete Automatic Flow

The sign-off run executes every stage in one command:

```bash
cd openlane
docker
./flow.tcl -design smart_parking -tag run3 -overwrite
```

<img width="1920" height="923" alt="final_layout_18" src="https://github.com/user-attachments/assets/d5ff48ae-6a25-47b9-8752-8010cf8e9ae1" />


Final summary of the sign-off run (`run3`):

| Metric | Value |
|---|---|
| Flow status | Flow completed without fatal errors |
| Cell count | 117 |
| Die area | 0.00798 mm² |
| Clock period | 10 ns |
| Magic DRC violations | 0 |
| LVS errors | 0 |
| Antenna violations | 0 |
| CVC errors | 0 |


<img width="1920" height="923" alt="results_9" src="https://github.com/user-attachments/assets/8ee36164-5863-457f-982c-237cde7c7074" />

## 9. Timing Constraints

Timing is evaluated with these constraints, which match the OpenLane synthesis setup:

```tcl
create_clock -name clk -period 10 [get_ports clk]

set_input_delay  2.0 -clock clk [all_inputs]
set_output_delay 2.0 -clock clk [all_outputs]

set_load 0.01765 [all_outputs]
```

| Constraint | Meaning |
|---|---|
| `create_clock` | Defines a 10 ns clock (100 MHz) |
| `set_input_delay` | Models a 2.0 ns external input delay (20 percent of the period) |
| `set_output_delay` | Applies a 2.0 ns output-delay requirement |
| `set_load` | Models the capacitive load on output ports |

The 10 ns period corresponds to a nominal clock frequency of 100 MHz. The multi-corner STA below uses an **ideal clock**, so CTS latency and skew are not included.

## 10. 16-Corner Static Timing Analysis (STA)

### 10.1 Why Multi-Corner STA Is Important

Standard-cell delay changes with process, supply voltage, and temperature. STA across PVT corners shows whether the design keeps its timing margin under different library conditions. Here, 16 SKY130 HD Liberty corners are applied to the post-route netlist and SPEF extracted from the final run.

```bash
sta -no_splash -exit corner.tcl   # run once per corner, with LIB and CORNER set
```

### 10.2 Results for All 16 Corners

| Corner | Setup slack (ns) | Hold slack (ns) | WNS (ns) | TNS (ns) | Status |
|---|---|---|---|---|---|
| ff_n40C_1v95 | 7.23 | 0.33 | 0.00 | 0.00 | PASS |
| ff_n40C_1v76 | 7.06 | 0.39 | 0.00 | 0.00 | PASS |
| ff_n40C_1v65 | 6.91 | 0.43 | 0.00 | 0.00 | PASS |
| ff_n40C_1v56 | 6.75 | 0.48 | 0.00 | 0.00 | PASS |
| ff_100C_1v95 | 7.22 | 0.36 | 0.00 | 0.00 | PASS |
| ff_100C_1v65 | 7.00 | 0.44 | 0.00 | 0.00 | PASS |
| tt_025C_1v80 | 6.66 | 0.52 | 0.00 | 0.00 | PASS |
| tt_100C_1v80 | 6.67 | 0.54 | 0.00 | 0.00 | PASS |
| ss_100C_1v60 | 5.08 | 1.03 | 0.00 | 0.00 | PASS |
| ss_100C_1v40 | 3.53 | 1.44 | 0.00 | 0.00 | PASS |
| ss_n40C_1v76 | 5.67 | 0.78 | 0.00 | 0.00 | PASS |
| ss_n40C_1v60 | 4.70 | 1.01 | 0.00 | 0.00 | PASS |
| ss_n40C_1v44 | 2.14 | 1.47 | 0.00 | 0.00 | PASS |
| ss_n40C_1v40 | 0.80 | 1.66 | 0.00 | 0.00 | PASS |

### 10.3 Standard Sign-Off Corners

| Corner | Setup slack | Hold slack |
|---|---|---|
| tt_025C_1v80 | 6.66 ns | 0.52 ns |
| ss_100C_1v60 | 5.08 ns | 1.03 ns |
| ff_n40C_1v95 | 7.23 ns | 0.33 ns |

All three standard corners meet setup and hold.

### TT Corner Result

<img width="1920" height="923" alt="tt1" src="https://github.com/user-attachments/assets/8753ae92-d3fd-436b-8528-94b888b12067" />

<img width="1920" height="923" alt="tt2" src="https://github.com/user-attachments/assets/194d7903-30a7-4d7f-af2a-875d702546d8" />


### SS Corner Result

<img width="1920" height="923" alt="tt3" src="https://github.com/user-attachments/assets/26c17c75-70d8-41b7-8d76-25e622a2dc50" />

<img width="1920" height="923" alt="tt4" src="https://github.com/user-attachments/assets/fde489c1-749b-4da8-adbf-524d51b5607b" />

### FF Corner Result

<img width="1920" height="923" alt="tt5" src="https://github.com/user-attachments/assets/b388e791-2ccc-48d0-8cbb-ada56fe7db02" />

### 10.4 Setup Slack Results

- **Maximum setup slack:** 7.23 ns (`ff_n40C_1v95`)
- **Minimum setup slack:** -6.77 ns (`ss_n40C_1v28`)
- 14 of 16 corners have positive setup slack.
- Two corners fail setup: `ss_n40C_1v35` (WNS -1.48 ns) and `ss_n40C_1v28` (WNS -6.77 ns). These are very low-voltage characterization corners (1.35 V and 1.28 V against a 1.80 V nominal supply), outside normal SKY130 sign-off conditions. The design meets setup at every corner of 1.40 V and above.

<img width="1920" height="923" alt="slack" src="https://github.com/user-attachments/assets/f26b0f95-8aa7-4236-9579-b67da0727fd7" />

<img width="1500" height="675" alt="image" src="https://github.com/user-attachments/assets/deeed5fb-b247-4902-839f-b62fbf988c99" />


### 10.5 Hold Slack Results

- **Minimum hold slack:** 0.33 ns (`ff_n40C_1v95`)
- **Maximum hold slack:** 2.62 ns (`ss_n40C_1v28`)
- All 16 corners have positive hold slack.

<img width="1920" height="923" alt="hold" src="https://github.com/user-attachments/assets/ec168139-a7a4-4101-84f2-83256877d970" />

<img width="1500" height="675" alt="image" src="https://github.com/user-attachments/assets/153dd433-0af5-4b5b-99f5-52de22bcb9b1" />

### 10.6 Setup and Hold Comparison, WNS and TNS


<img width="1920" height="923" alt="comp" src="https://github.com/user-attachments/assets/ce321a13-7590-4d97-a5ce-8e1abbb7b4dd" />


<img width="1920" height="923" alt="wns and tns" src="https://github.com/user-attachments/assets/b45891c6-1576-4528-a242-4c4885cca92f" />

<img width="1500" height="675" alt="image" src="https://github.com/user-attachments/assets/dc9ed761-9e94-4cc9-9e1f-a01c1e20859c" />


<img width="1500" height="675" alt="image" src="https://github.com/user-attachments/assets/a6962476-e244-448d-be89-b35981e82dce" />


<img width="1500" height="675" alt="image" src="https://github.com/user-attachments/assets/53354863-c69a-41b9-ac49-3b2b1f9df606" />

The full per-corner worksheet is in `sta/smart_parking_sta_corners.xlsx`, and the complete path reports are in `sta/reports/`.

### 10.7 Interpretation

Setup slack shrinks as the supply voltage drops at the slow corners. At nominal and high-voltage corners the critical setup path is the input path from `vehicle_entry` to a state flip-flop. At the low-voltage slow corners the critical path becomes a flip-flop to flip-flop path through several mux and gate stages. Hold slack is smallest at the fast corners, as expected, but remains positive everywhere.

These results apply to the stated constraints with an ideal clock. A complete sign-off should also review CTS skew and latency, all path groups, and any tool warnings.

## 11. Physical Verification: DRC, LVS, and Antenna

### 11.1 Design Rule Checking (DRC)

```text
Magic DRC Summary
Total Magic DRC violations is 0
```

**Result: 0 Magic DRC violations reported.**

<img width="1920" height="923" alt="magic" src="https://github.com/user-attachments/assets/06e0bc75-0b8f-4e2a-94ea-f447da6d1594" />


### 11.2 Layout Versus Schematic (LVS)

```text
LVS Summary
LVS reports no net, device, pin, or property mismatches.
Total errors = 0
```

**Result: 0 LVS errors reported.**


<img width="1920" height="923" alt="magic1" src="https://github.com/user-attachments/assets/1ad9fcd3-d06f-49db-a137-8bf429266339" />

### 11.3 Antenna Check

**Result: 0 antenna violations reported.**

### 11.4 Verification Scope

- DRC evaluates layout geometry against the design rules.
- LVS evaluates layout-to-netlist correspondence.
- STA evaluates timing under the stated constraints and analysis conditions.

Passing one category does not prove that every other category passes.

## 12. Final Physical Layout and GDSII

The final GDSII database contains the physical layout generated by the flow. It can be inspected with Magic:

```bash
magic -T $PDK_ROOT/sky130A/libs.tech/magic/sky130A.tech \
  gds read results/magic/smart_parking_controller.gds
```

### Final GDS Layout

<img width="1920" height="923" alt="gds1" src="https://github.com/user-attachments/assets/eca293d5-79fa-4be8-a9d7-7ea9d2909403" />


<img width="1920" height="923" alt="gds" src="https://github.com/user-attachments/assets/4549820e-5134-416f-8315-889fe73869b4" />

## 13. Repository Structure

```text
smart-parking-controller/
├── README.md
├── config.json
├── src/
│   ├── smart_parking_controller.v
│   ├── tb_smart_parking_controller.v
│   ├── rtl_sim.log
│   └── smart_parking.vcd
├── gls/
│   ├── tb_gls.v
│   ├── smart_parking_controller.synthesis.v
│   ├── used_models.v
│   ├── gls_sim.log
│   ├── rtl_vs_gls.diff
│   └── smart_parking_gls.vcd
├── sta/
│   ├── corner.tcl
│   ├── smart_parking_sta_corners.xlsx
│   └── reports/
│       └── sta_<corner>.rpt          (16 files)
├── documentation/
│   ├── simulation/
│   ├── synthesis/
│   │   └── 1-yosys_4.stat.rpt
│   ├── gls/
│   ├── sta/
│   ├── layout/
│   └── physical_verification/
│       └── final_summary_report.csv
└── .gitignore
```

## 14. How to Reproduce

```bash
# 1. RTL simulation
cd src
iverilog -o rtl_sim smart_parking_controller.v tb_smart_parking_controller.v
vvp rtl_sim | tee rtl_sim.log

# 2. Synthesis and physical design (OpenLane container)
cd openlane
docker
./flow.tcl -design smart_parking -tag run3 -overwrite

# 3. Results
cat designs/smart_parking/runs/run3/reports/final_summary_report.csv
```

`config.json`:

```json
{
  "DESIGN_NAME": "smart_parking_controller",
  "VERILOG_FILES": "$::env(DESIGN_DIR)/src/smart_parking_controller.v",
  "CLOCK_PORT": "clk",
  "CLOCK_PERIOD": 10.0,
  "FP_CORE_UTIL": 20,
  "PL_TARGET_DENSITY": 0.3
}
```

## 15. Final Results Summary

| Item | Reported result |
|---|---|
| RTL design | Verilog sequential controller, 8 slots |
| FSM | 4 states |
| RTL simulation | 10 of 10 tests pass |
| Gate-level simulation | 10 of 10 tests pass, outputs match RTL |
| Clock period | 10 ns (100 MHz) |
| Synthesis cell count | 117 |
| Synthesis area | 1078.53 µm² |
| Die area | 0.00798 mm² |
| STA corners | 16 |
| Setup slack range | -6.77 ns to 7.23 ns (14 of 16 corners pass) |
| Hold slack range | 0.33 ns to 2.62 ns (all corners pass) |
| Standard corners (TT, SS, FF) | All pass setup and hold |
| Magic DRC | 0 violations |
| LVS | 0 errors |
| Antenna | 0 violations |

## 16. Conclusion

This project takes a smart parking controller from Verilog RTL through simulation, synthesis, gate-level simulation, and physical implementation using an open-source flow and the SKY130 HD library.

The design uses a four-state FSM and a 4-bit occupancy counter for 8 slots. RTL and gate-level simulation both pass all 10 test scenarios with matching outputs. The final layout reports zero DRC violations, zero LVS errors, and zero antenna violations. Timing is met at all standard corners and at 14 of the 16 evaluated PVT corners. The two failing corners are extreme low-voltage corners outside normal sign-off conditions.

The project brings RTL design, verification, synthesis, physical implementation, multi-corner timing analysis, and layout verification together in one documented hardware-design workflow.

**Project:** Design and Physical Implementation of a Smart Parking Management Controller  
**HDL:** Verilog  
**Technology:** SKY130 HD  
**Implementation flow:** OpenLane / OpenROAD  
**Verification:** RTL simulation, GLS, multi-corner STA, Magic DRC, LVS


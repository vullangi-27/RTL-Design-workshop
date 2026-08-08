# RTL Design Workshop

Welcome to my **RTL Design Workshop** repository.

This repository contains my learning progress, experiments, and documentation related to **Verilog RTL Design and Simulation**. Each day contains the concepts learned, practical experiments performed, simulation results, and observations.

---

## 📚 Workshop Progress

| Day | Topic | Status |
|-----|-------|--------|
| Day 1 | Introduction to Verilog RTL Design & Simulation | ✅ Completed |

---

## 📂 Repository Structure

```
RTL_Workshop
│
├── README.md
│
└── Day_1
    └── README.md
```

---

# Day 1 – Introduction to Verilog RTL Design & Simulation

In Day 1, I explored the fundamentals of **RTL design using Verilog** and learned the basic simulation workflow.

### Topics Covered

- Understanding Simulator, Design, and Testbench
- Introduction to Icarus Verilog (iverilog)
- Verilog simulation flow
- Implementation and simulation of a 2:1 Multiplexer
- Waveform analysis using GTKWave

---

## 🔗 Day 1 Documentation


# Day 1 – Exploring Verilog RTL Design Through Simulation

## Experiment Objective

The objective of this experiment was to understand the fundamentals of Register Transfer Level (RTL) design using **Verilog**. The experiment also focused on learning how to compile and simulate Verilog designs using **Icarus Verilog (iverilog)** and verify the output through waveform analysis in **GTKWave**. A **2-to-1 Multiplexer** was implemented to understand the complete simulation process.

---

##  Contents

- Digital Design Verification
- Simulation Workflow with Icarus Verilog
- Practical Exercise – Simulating a 2:1 Multiplexer
- Multiplexer Design Explanation
- Introduction to Yosys
- Conclusion

---

## Digital Design Verification

### Simulator

A **simulator** is a software application that executes a Verilog design in a virtual environment to evaluate its behavior. It allows designers to observe how a digital circuit responds to different input conditions and helps identify functional errors before hardware implementation.

---

### Design

The **design** is the Verilog module that represents the digital circuit to be implemented. It defines the circuit's logic, specifying how inputs are processed to produce the desired outputs.

---

### Testbench

A **testbench** is a dedicated verification module written to test the functionality of a design. It applies different combinations of input signals, monitors the resulting outputs, and helps confirm that the design performs according to its intended behavior.


<img width="606" height="285" alt="testbench" src="https://github.com/user-attachments/assets/8f3425fa-8804-4b28-b919-38ae2d3fdfc4" />

---

# 2️⃣ Simulation Workflow with Icarus Verilog

**Icarus Verilog (iverilog)** is an open-source Verilog compiler and simulator. It compiles the design and testbench, executes the simulation, and generates a **Value Change Dump (.vcd)** file that can be viewed in **GTKWave**.

## Simulation Flow

```text
Design File
      +
Testbench
      ↓
Icarus Verilog (iverilog)
      ↓
Generate .vcd File
      ↓
GTKWave
```

### Simulation Flow Diagram

<img width="701" height="325" alt="simflow" src="https://github.com/user-attachments/assets/fcc78909-5229-4f89-b4c3-58c8cd441f4f" />


---

# 3️⃣ Practical Exercise – Simulating a 2:1 Multiplexer

## Step 1 – Install Required Tools

```bash
sudo apt install iverilog
sudo apt install gtkwave
```

---

## Step 2 – Compile the Design

```bash
iverilog good_mux.v tb_good_mux.v
```

This command compiles the design file and the testbench.

---

## Step 3 – Execute the Simulation

```bash
./a.out
```

Running the above command executes the simulation and generates the waveform file.

---

## Step 4 – Open the Waveform

```bash
gtkwave tb_good_mux.vcd
```

The waveform can now be analyzed using GTKWave.

### GTKWave Output
<img width="1599" height="899" alt="WhatsApp Image 2026-08-08 at 4 44 34 PM" src="https://github.com/user-attachments/assets/af1a9210-dc7c-4560-a29f-076a322ca7d1" />



---

# 4️⃣ Multiplexer Design Explanation

## Verilog Design

```verilog
module good_mux (
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(*)
begin
    if (sel)
        y <= i1;
    else
        y <= i0;
end

endmodule
```

---

## Working Principle

### Inputs

- `i0` – First input
- `i1` – Second input
- `sel` – Selection signal

### Output

- `y` – Multiplexer output

### Operation

- When `sel = 0`, the output follows **i0**.
- When `sel = 1`, the output follows **i1**.

### Verilog Code Screenshot
<img width="1600" height="769" alt="image" src="https://github.com/user-attachments/assets/420da0ee-02b6-4004-b7fb-0540f1447e67" />



---

# 5️⃣ Introduction to Yosys

**Yosys** is an open-source tool used to synthesize Verilog RTL designs and generate a gate-level netlist.

### Yosys Synthesis Flow

The basic synthesis flow is:

1. Load the technology library using `read_liberty`.
2. Read the RTL design using `read_verilog`.
3. Set the top module using `synth -top`.
4. Perform technology mapping using `abc`.
5. Generate the netlist using `write_verilog`.

```text
RTL Design + Library
        ↓
      Yosys
        ↓
Synthesized Netlist
```

```bash
read_liberty -lib <library>.lib
read_verilog good_mux.v
synth -top good_mux
abc -liberty <library>.lib
write_verilog synthesized_mux.v
```

The synthesized netlist is then verified using the testbench and **GTKWave**.

<img width="705" height="416" alt="Screenshot 2026-08-08 222843" src="https://github.com/user-attachments/assets/c07f8c9e-fe0f-4822-881d-e827b5b9f352" />
<img width="703" height="334" alt="Screenshot 2026-08-08 222858" src="https://github.com/user-attachments/assets/3eff0420-b3f9-4bb9-a830-da05794bb501" />



# 6️⃣ Conclusion

Through this experiment, I learned the basic RTL design flow using Verilog. I understood the purpose of a simulator, design, and testbench, successfully compiled and simulated a **2:1 Multiplexer** using **Icarus Verilog**, and verified the circuit's functionality using **GTKWave** and an introduction to **Yosys synthesis**. This experiment provided a strong foundation for further digital design experiments.

---
---

## 🛠 Tools Used

- Verilog
- Icarus Verilog (iverilog)
- GTKWave

---


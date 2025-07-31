# Physical Design of a RISC-V Core with OpenLANE & Sky130 PDK

### 📘 Project Overview

This project documents the complete physical design (PD) flow for a `picorv32a` RISC-V core, from RTL to a final GDSII layout. It leverages the fully automated, open-source **OpenLANE** framework and the **Skywater 130nm (Sky130) PDK**. This work was completed as part of the Digital VLSI SoC Design and Planning Workshop by VLSI System Design (VSD) Corp, providing a practical, hands-on journey through the entire chip implementation process.

OpenLANE orchestrates a suite of powerful EDA tools—including Yosys, OpenROAD, Magic, and Netgen—to automate the entire flow, from logic synthesis to final layout generation, enabling the efficient creation of hard macros and full chip designs.

---

### 📁 Repository Contents

- [**Day 1: Getting Started with Open-Source EDA**](#-day-1-getting-started-with-open-source-eda)
  - Theory: IC Components, RISC-V ISA, RTL-to-GDSII Flow
  - Lab: Synthesis, Flip-Flop Ratio, Timing Slack Analysis
- [**Day 2: Floorplanning & Placement**](#-day-2-floorplanning--placement)
  - Theory: Floorplanning Principles, Placement Strategies
  - Lab: Core & Die Layout, Cell Placement, Library Characterization
- [**Day 3: Standard Cell Design & Characterization**](#-day-3-standard-cell-design--characterization)
  - Theory: Cell Design, SPICE Simulation, 16-Mask CMOS Process
  - Lab: Layout Extraction, Post-Layout Simulation, DRC Correction
- [**Day 4: Clock Tree Synthesis & Timing**](#-day-4-clock-tree-synthesis--timing)
  - Theory: Delay Models, Timing Analysis, CTS
  - Lab: Integrating Custom Cells, Pre-Layout & Post-CTS Analysis
- [**Day 5: Routing & Final Sign-off**](#-day-5-routing--final-sign-off)
  - Lab: Power Distribution, Routing, Parasitic Extraction, GDSII Generation

---

### 🚀 Day 1: Getting Started with Open-Source EDA

This day covers the foundational concepts of IC design and the OpenLANE flow.

#### Theory

**Anatomy of an IC**
An Integrated Circuit is composed of several key physical elements:
* **Package:** The protective casing (e.g., QFN-48) that houses the silicon and connects it to the PCB.
* **Die:** The raw silicon chip where transistors are fabricated.
* **Core:** The central functional area of the die containing the digital logic.
* **Pads:** Electrical contact points that connect the core's signals to the package pins.
* **Foundry IPs:** Specialized blocks (e.g., PLLs, ADCs) designed by the foundry that require analog expertise.
* **Macros:** Purely digital logic blocks (e.g., RISC-V SoC, SPI) that are integrated into the core.

<img width="848" height="853" alt="182751377-2810d388-21b0-4df1-b1d4-c72176d80d28" src="https://github.com/user-attachments/assets/d6c9f443-dc08-4587-972a-c46c9b927dad" />

**Introduction to RISC-V**
RISC-V is an open-standard Instruction Set Architecture (ISA) built on the principles of Reduced Instruction Set Computing. Its key features are:
* **Open-Source:** Free to use, modify, and implement without licensing fees.
* **Modularity:** Consists of a base integer ISA with optional extensions for multiplication, floating-point, etc.
* **Simplicity & Scalability:** Ideal for a wide range of applications, from embedded systems to supercomputers.

**The RTL-to-GDSII Flow**
This is the standard process for converting a hardware description into a physical layout ready for fabrication.
1.  **Synthesis:** Converts RTL code (Verilog/VHDL) into a gate-level netlist using standard cells from the PDK library.
2.  **Floorplanning:** Defines the chip's physical hierarchy, including the die size, core area, I/O pad placement, and power grid planning.
3.  **Placement:** Determines the precise physical location of each standard cell in the core, optimizing for wirelength and congestion.
4.  **Clock Tree Synthesis (CTS):** Builds a buffered clock network to distribute the clock signal to all flip-flops with minimal skew and jitter.
5.  **Routing:** Connects the cell pins with metal wires according to the netlist, using the multiple metal layers available in the PDK.
6.  **Verification:** Before "tape-out" (sending the design to the foundry), the layout is checked for:
    * **DRC (Design Rule Check):** Ensures the layout meets the foundry's manufacturing rules.
    * **LVS (Layout Versus Schematic):** Confirms the layout is electrically identical to the original schematic.
    * **STA (Static Timing Analysis):** Verifies that the design meets all timing constraints.

The final output is a **GDSII** file, a database that describes the physical layout of the chip.

<img width="848" height="=500" alt="182759711-6b9352ec-7652-4589-af31-53a409eb2830" src="https://github.com/user-attachments/assets/f3cdde11-73e1-496e-9486-3d02f62ff65d"/>

Inside a specific design folder, a `config.tcl` file overrides the default settings in OpenLANE.

<img width="848" height="126" alt="Screenshot 2025-07-24 142941" src="https://github.com/user-attachments/assets/a61e7285-ca8d-45ef-adc8-2522e4e1103e" />

#### Lab Work

**Synthesis**
The lab begins by launching OpenLANE in interactive mode and preparing the design.
* **Launch OpenLANE:**
    <img width="848" height="250" alt="Screenshot 2025-07-24 120932" src="https://github.com/user-attachments/assets/42c4e8a2-9a39-4ac9-9a62-2be252e3fb0f" />
* **Prepare Design:** The `prep -design picorv32a` command sets up the run directory and merges the necessary LEF files.
    <img width="848" height="370" alt="Screenshot 2025-07-24 121111" src="https://github.com/user-attachments/assets/d9540d85-6019-4535-8d9a-02e1ee24831c" />
* **Run Synthesis:** The `run_synthesis` command executes Yosys to synthesize the RTL into a gate-level netlist. After running, results and reports are generated.
    <img width="848" height="107" alt="Screenshot 2025-07-24 132349" src="https://github.com/user-attachments/assets/a0e2bb41-4784-4395-8c2a-7c3d3f53156a" />
    <img width="848" height="130" alt="Screenshot 2025-07-24 133023" src="https://github.com/user-attachments/assets/78de66bd-0091-43ba-9f40-47aad4db611c" />

**Flip-Flop Ratio Estimation**
<img width="848" height="600" alt="Screenshot 2025-07-24 123628" src="https://github.com/user-attachments/assets/30b7898a-9a64-4365-9e04-f5027549eead" />

The flip-flop to total cell ratio is a useful metric for design complexity.
* Number of D-type flip-flops (`sky130_fd_sc_hd__dff_...`): 1613
* Total number of cells: 14876
* **Flip-Flop Ratio:** $1613 / 14876 \approx 10.84\%$

**Fixing Timing Slack**
**Slack** is the margin by which a design meets its timing constraints. Negative slack means the circuit is too slow.
* **Before:** The initial run had negative slack.
    <img width="848" height="255" alt="Screenshot 2025-07-24 123323" src="https://github.com/user-attachments/assets/9fe5c38e-a80a-4875-9cfc-4d0b81cf11c3" />
* **After:** The clock period was relaxed to `55.00` to achieve positive slack.
    <img width="848" height="255" alt="Screenshot 2025-07-24 131231" src="https://github.com/user-attachments/assets/ae72bbe7-8a79-40b1-80b4-6af999df93fb" />

---

### 🛠️ Day 2: Floorplanning & Placement

This day focuses on creating the physical layout of the chip core and placing the standard cells.

#### Theory

**Floorplanning**
Floorplanning is the blueprint for the chip's layout.
* **Core Utilization & Aspect Ratio**
    <img width="848" height="367" alt="Screenshot 2025-07-24 223107" src="https://github.com/user-attachments/assets/92da74da-1a22-40b1-9b2e-09e5d11c03c7" />
* **Pre-placed Cells**
    <img width="848" height="367" alt="Screenshot 2025-07-24 223127" src="https://github.com/user-attachments/assets/675fe99d-0725-4d39-ab18-e5d19f703447" />
* **Decoupling Capacitors (Decaps)**
    <img width="848" height="603" alt="Untitled" src="https://github.com/user-attachments/assets/c04cf8d8-f3de-49f7-bda0-91d5892b8fe5" />
* **Power Planning**
    <img width="848" height="371" alt="Screenshot 2025-07-24 222913" src="https://github.com/user-attachments/assets/4c943466-20cc-4e48-96ca-6c0bcdba10ab" />
* **Pin Placement & Blockages**
    <img width="848" height="355" alt="Screenshot 2025-07-24 222958" src="https://github.com/user-attachments/assets/aa9bb6ac-70b1-4a1e-a388-22b122db452f" />
    <img width="848" height="347" alt="Screenshot 2025-07-24 223040" src="https://github.com/user-attachments/assets/0f4ab5a9-2147-4725-8bc1-e3412078ac7b" />

**Placement**
During placement, the standard cells from the netlist are assigned physical locations within the core rows.
* **Global Placement:** An initial, approximate placement that optimizes for wirelength and congestion.
* **Detailed Placement:** Refines the global placement to ensure all cells are legally placed on the grid without overlaps.

#### Lab Work

**Floorplanning in OpenLANE**
The `run_floorplan` command generates the initial layout of the die and core.
* **Viewing the result in Magic:**
    <img width="848" height="37" alt="Screenshot 2025-07-24 134349" src="https://github.com/user-attachments/assets/40cb2aba-9fc6-43e8-a196-1216bfb0aa48" />
    <img width="848" height="732" alt="Screenshot 2025-07-24 133352" src="https://github.com/user-attachments/assets/b7d0b4e7-4386-4b35-90fe-e37f92b2afa1" />
    <img width="848" height="907" alt="Screenshot 2025-07-24 133602" src="https://github.com/user-attachments/assets/c546ebab-bfb0-4631-9967-62575694aa4a" />
    <img width="848" height="668" alt="Screenshot 2025-07-24 133654" src="https://github.com/user-attachments/assets/770e1f71-f57d-44e4-b710-31d5bbab4fac" />

**Placement in OpenLANE**
The `run_placement` command places the standard cells into the core rows.
* **Viewing the result in Magic:**
    <img width="848" height="42" alt="Screenshot 2025-07-24 134409" src="https://github.com/user-attachments/assets/35373e18-0d02-4666-9629-b5e8f4adcdc3" />
    <img width="848" height="728" alt="Screenshot 2025-07-24 134153" src="https://github.com/user-attachments/assets/10d3ff16-2da9-4f14-a763-c834a3e04b00" />
    <img width="848" height="545" alt="Screenshot 2025-07-24 134259" src="https://github.com/user-attachments/assets/dc7bd71a-8625-4cd5-abad-92c53610c52d" />

**Library Characterization**
<img width="848" height="603" alt="dadda" src="https://github.com/user-attachments/assets/a288d9cd-e1cc-4df1-bdf8-d5c77d97a7c4" />

**Estimating Die Area**
The die area is defined in the `picorv32a.floorplan.def` file.
<img width="848" height="107" alt="Screenshot 2025-07-24 132349" src="https://github.com/user-attachments/assets/472471ea-1a17-4a3b-8d65-77c46b1f1ffb" />
<img width="848" height="592" alt="Screenshot 2025-07-24 132311" src="https://github.com/user-attachments/assets/2cdf5fe9-7670-4eb5-a1dd-8c9a8d98e253" />
* **Area Calculation:** $(660.685 \mu m) \times (671.405 \mu m) \approx 443,587 \mu m^2$.

---

### 🎨 Day 3: Standard Cell Design & Characterization

This day delves into the design and simulation of a basic CMOS inverter, a fundamental building block.

#### Theory

**Designing a Library Cell**
Creating a standard cell involves circuit design, transistor sizing, layout, and characterization.
<img width="848" height="602" alt="Screenshot 2025-07-26 125422" src="https://github.com/user-attachments/assets/f647f49c-e4af-4a20-9eaf-dfa6f4d9f203" />

**Simulation in ngspice**
<img width="848" height="656" alt="Screenshot 2025-07-26 125706" src="https://github.com/user-attachments/assets/4e34ffd1-4c48-4ab0-b0fd-cba749e4df5a" />
<img width="848" height="648" alt="Screenshot 2025-07-26 125749" src="https://github.com/user-attachments/assets/a225d17d-3e87-4ea4-a6d9-4e5caf95988f" />

**SPICE Switching Threshold and Propagation Delay**
* Switching Threshold
    <img width="848" height="541" alt="Screenshot 2025-07-26 135748" src="https://github.com/user-attachments/assets/d3abddf5-db6a-448f-952e-b885b8568468" />
* Propagation Delay (Transient Analysis)
    <img width="200" height="216" alt="Screenshot 2025-07-26 135659" src="https://github.com/user-attachments/assets/7ab2839f-d176-49b6-b1b5-30f36c04c5e2" />
    <img width="848" height="53" alt="Screenshot 2025-07-26 135650" src="https://github.com/user-attachments/assets/957730e3-5601-4bbc-a6d9-46503961e79a" />
    <img width="848" height="889" alt="187056370-18949899-a158-4307-96d9-d5c06bbeed66" src="https://github.com/user-attachments/assets/754e0757-5b33-494c-95ae-2a287599dc6d" />

**16-Mask CMOS Process**
This is a simplified overview of the steps to fabricate a CMOS inverter.
<img width="848" height="400" alt="Screenshot 2025-07-26 174207" src="https://github.com/user-attachments/assets/fa5ac8b5-675a-4151-a230-072c64d6137e" />

#### Lab Work

**SPICE Extraction of Inverter in Magic**
<img width="848" height="52" alt="Screenshot 2025-07-26 174607" src="https://github.com/user-attachments/assets/b185ee77-7baf-4d9d-a14e-8f3ce328d139" />
<img width="848" height="168" alt="Screenshot 2025-07-30 125827" src="https://github.com/user-attachments/assets/73cde73d-b8ce-40b6-b346-73d279bccbc6" />
<img width="848" height="57" alt="Screenshot 2025-07-26 174719" src="https://github.com/user-attachments/assets/144921cd-5e35-4c8b-9918-89460b008333" />
<img width="848" height="260" alt="Screenshot 2025-07-26 174858" src="https://github.com/user-attachments/assets/76fb85c3-e007-4032-9925-8800b238c3e0" />
<img width="848" height="721" alt="Screenshot 2025-07-26 172928" src="https://github.com/user-attachments/assets/9d946e89-dce5-46dd-be89-e8f93b824935" />

**Post-Layout SPICE simulation (ngspice)**
<img width="848" height="875" alt="Screenshot 2025-07-26 181256" src="https://github.com/user-attachments/assets/6b11a52d-e7dd-4626-882e-e8d24f331810" />
<img width="848" height="487" alt="Screenshot 2025-07-26 181156" src="https://github.com/user-attachments/assets/fa7d1c08-da3b-4118-83f7-42ff78559790" />

**Slew Rate and Propagation Delay**
* **Rise Transition Time:**
    <img width="848" height="78" alt="Screenshot 2025-07-26 213748" src="https://github.com/user-attachments/assets/02b52e5c-fff7-4283-8258-e47d702eabac" />
* **Fall Transition Time:**
    <img width="848" height="77" alt="Screenshot 2025-07-26 214030" src="https://github.com/user-attachments/assets/b9a32895-a01c-4740-9f63-48f32acf497e" />
* **Rise Propagation Delay:**
    <img width="848" height="72" alt="Screenshot 2025-07-26 214545" src="https://github.com/user-attachments/assets/158e2eaf-4f69-4fbd-82bb-5b2c5814962d" />

**Fixing DRC in the Tech File**
<img width="848" height="63" alt="Screenshot 2025-07-26 232040" src="https://github.com/user-attachments/assets/102cbe43-321e-451d-8e0e-82f87013bd23" />
<img width="848" height="472" alt="Screenshot 2025-07-26 221828" src="https://github.com/user-attachments/assets/48b3df34-f795-4d5b-9c47-09462b66b709" />
<img width="848" height="532" alt="Screenshot 2025-07-26 223639" src="https://github.com/user-attachments/assets/d9732a2e-a38e-49a6-9374-ab0a0183dd06" />
<img width="400" height="137" alt="Screenshot 2025-07-26 232204" src="https://github.com/user-attachments/assets/d39c2834-fb34-45d8-8f2f-93e4ffb25187" />
<img width="400" height="193" alt="Screenshot 2025-07-26 223705" src="https://github.com/user-attachments/assets/db2ff8f4-d82f-454b-bf15-88568d65e412" />
<img width="400" height="151" alt="Screenshot 2025-07-26 223718" src="https://github.com/user-attachments/assets/e36aa647-54e9-460a-bd63-e89fcb35c495" />
<img width="400" height="87" alt="Screenshot 2025-07-26 223505" src="https://github.com/user-attachments/assets/a6ec936c-e03e-440c-9729-59cd987577ab" />

---

### ⏱️ Day 4: Clock Tree Synthesis & Timing

This day focuses on creating a robust clock network and analyzing timing with a custom-designed cell.

#### Theory

**Delay Table**
<img width="1166" height="635" alt="Screenshot 2025-07-30 223725" src="https://github.com/user-attachments/assets/15e6a690-8f95-41c5-8027-a0e04d836538" />

**Clock Tree Synthesis (CTS)**
<img width="676" height="457" alt="Screenshot 2025-07-30 224543" src="https://github.com/user-attachments/assets/765ab7fb-20ea-40a2-a724-032cb203e7ad" />

#### Lab Work

**Tracks and Grids**
<img width="848" height="496" alt="Screenshot 2025-07-30 000535" src="https://github.com/user-attachments/assets/5b0f8ca6-d1d9-4205-afaa-3d4e0c39272e" />
<img width="848" height="490" alt="Screenshot 2025-07-30 000549" src="https://github.com/user-attachments/assets/bbf1a286-b4cd-44bf-a572-249b2c87fe0e" />
<img width="848" height="417" alt="Screenshot 2025-07-30 113158" src="https://github.com/user-attachments/assets/b6e985f8-a29b-477c-806a-c1da7bda2ab1" />
<img width="848" height="848" alt="Screenshot 2025-07-28 121258" src="https://github.com/user-attachments/assets/cc10810e-5776-436d-a05d-44b4b84273b9" />
<img width="848" height="608" alt="Screenshot 2025-07-27 011229" src="https://github.com/user-attachments/assets/5c5bc380-b802-41c4-b1a4-0f610579fcf9" />

**Integrating a Custom Inverter**
<img width="848" height="146" alt="Screenshot 2025-07-27 015605" src="https://github.com/user-attachments/assets/ae0c769f-27b8-4342-84ce-398afc2b430d" />
<img width="848" height="145" alt="Screenshot 2025-07-30 130455" src="https://github.com/user-attachments/assets/a75274fc-1505-4900-a4a6-2f0cd839281c" />
<img width="848" height="292" alt="Screenshot 2025-07-27 112459" src="https://github.com/user-attachments/assets/cd356fb9-25aa-4665-b87b-1e17dc3b5659" />
<img width="848" height="267" alt="Screenshot 2025-07-30 113419" src="https://github.com/user-attachments/assets/5022ddbe-00f2-4f35-babc-350cc474992e" />
<img width="848" height="582" alt="Screenshot 2025-07-30 004441" src="https://github.com/user-attachments/assets/db94ac64-9f76-400c-83bc-161d92eb155d" />
<img width="848" height="902" alt="Screenshot 2025-07-30 131638" src="https://github.com/user-attachments/assets/516d4ac8-968c-4285-9784-cfb003c45fed" />
<img width="1697" height="892" alt="Screenshot 2025-07-30 132028" src="https://github.com/user-attachments/assets/647d1896-6a85-48f4-92e6-a4985aa4c76c" />

**Optimizing Synthesis**
<img width="848" height="481" alt="Screenshot 2025-07-28 130724" src="https://github.com/user-attachments/assets/bd6f4d92-62b0-427f-badd-2cc9968b03a4" />
<img width="848" height="387" alt="Screenshot 2025-07-30 115338" src="https://github.com/user-attachments/assets/7a3485fb-6d89-40cc-84f6-17800e613640" />
<img width="848" height="897" alt="Screenshot 2025-07-28 222817" src="https://github.com/user-attachments/assets/84f09731-1370-4d3c-b9c8-409273ba04f7" />
<img width="848" height="497" alt="Screenshot 2025-07-28 134549" src="https://github.com/user-attachments/assets/c044a361-b554-41d6-a30f-f4d218789c12" />
<img width="848" height="921" alt="Screenshot 2025-07-28 223452" src="https://github.com/user-attachments/assets/354f77d9-484b-4fc9-82db-f98ba83e714f" />
<img width="848" height="905" alt="Screenshot 2025-07-28 225050" src="https://github.com/user-attachments/assets/8fd2aa54-a14f-4a08-9c63-2e0da79a209a" />

**Running CTS & Post-CTS Analysis**
<img width="1847" height="883" alt="Screenshot 2025-07-28 225444" src="https://github.com/user-attachments/assets/9c38515d-dcd4-43cb-93d8-db4e18e8d09e" />
<img width="1847" height="883" alt="Screenshot 2025-07-28 225444" src="https://github.com/user-attachments/assets/2e5e7292-a69f-4646-9e8a-3f35f27d626a" />
<img width="1832" height="245" alt="Screenshot 2025-07-30 124909" src="https://github.com/user-attachments/assets/cd4713c1-b98b-4fcd-b906-30b1b344b2a8" />
<img width="1438" height="740" alt="Screenshot 2025-07-30 125132" src="https://github.com/user-attachments/assets/db8a2104-c693-4c8c-ae57-b77b073c9139" />
<img width="1441" height="738" alt="Screenshot 2025-07-29 010515" src="https://github.com/user-attachments/assets/3d0ac85a-1153-45a3-aff5-97221c0888b9" />
<img width="1505" height="782" alt="Screenshot 2025-07-29 010553" src="https://github.com/user-attachments/assets/9461a51c-4038-4e8f-ad4a-088adf6a8967" />
<img width="1297" height="457" alt="Screenshot 2025-07-29 181220" src="https://github.com/user-attachments/assets/1cb63874-68e5-4fba-a84f-a27dcec9eb0b" />
<img width="1847" height="272" alt="Screenshot 2025-07-29 181247" src="https://github.com/user-attachments/assets/7573ef66-934f-45e7-8193-366e2c699ff4" />
<img width="1257" height="480" alt="Screenshot 2025-07-29 181627" src="https://github.com/user-attachments/assets/4123c22c-b05e-457c-a5f0-804acaad7fc8" />
<img width="1841" height="620" alt="Screenshot 2025-07-29 181808" src="https://github.com/user-attachments/assets/7d40a9ad-701b-4e1f-8449-5c060cdf403c" />
<img width="1847" height="908" alt="Screenshot 2025-07-29 181832" src="https://github.com/user-attachments/assets/2f8ed163-c532-491e-906b-cad0c448fc34" />
<img width="1830" height="878" alt="Screenshot 2025-07-29 181916" src="https://github.com/user-attachments/assets/1e754cbe-4c97-479c-bcce-e9beb3ce3154" />
<img width="1852" height="907" alt="Screenshot 2025-07-29 182340" src="https://github.com/user-attachments/assets/e4d7ed6d-9bc7-4d7a-975e-4a574397192c" />
<img width="1518" height="655" alt="Screenshot 2025-07-30 222049" src="https://github.com/user-attachments/assets/3a085a95-c2b1-493c-8b05-0ca7bbe19744" />
<img width="1848" height="916" alt="Screenshot 2025-07-29 183123" src="https://github.com/user-attachments/assets/d270f744-9aec-439a-b537-3e5fb5a04a9a" />
<img width="1337" height="740" alt="Screenshot 2025-07-30 220638" src="https://github.com/user-attachments/assets/f89677be-3220-45dc-8868-1bde680a25e1" />
<img width="1852" height="945" alt="Screenshot 2025-07-29 183410" src="https://github.com/user-attachments/assets/5b3b741f-fc9e-40d4-9720-3ff3ae5cf279" />

---

### 🏁 Day 5: Routing & Final Sign-off

The final day completes the physical design flow with power grid generation, routing, and final verification.

#### Lab Work

**Power Distribution Network (PDN)**
<img width="1845" height="898" alt="Screenshot 2025-07-29 183947" src="https://github.com/user-attachments/assets/0623ac22-6ec7-4ea5-9456-c67c1de65eab" />
<img width="1852" height="906" alt="Screenshot 2025-07-29 184133" src="https://github.com/user-attachments/assets/944c3ba9-5daf-4bd0-8a20-96b47da0d1c6" />
<img width="1810" height="887" alt="Screenshot 2025-07-29 184343" src="https://github.com/user-attachments/assets/23dc0a50-d4e4-4b89-8e1e-8fab538c8f30" />
<img width="1847" height="902" alt="Screenshot 2025-07-29 184540" src="https://github.com/user-attachments/assets/f948ace1-3fe0-488a-9a35-a88aca1b8b36" />
<img width="1856" height="898" alt="Screenshot 2025-07-29 184602" src="https://github.com/user-attachments/assets/4d1d219b-4dc2-4011-9633-dfe76a9c24be" />
<img width="1606" height="818" alt="Screenshot 2025-07-29 184757" src="https://github.com/user-attachments/assets/d80cc966-ecb3-4adc-a923-1d4fe65a3cc7" />
<img width="1502" height="838" alt="Screenshot 2025-07-29 184832" src="https://github.com/user-attachments/assets/8979c7cf-74b0-4249-b8ba-4a8595d78207" />
<img width="1506" height="837" alt="Screenshot 2025-07-29 184913" src="https://github.com/user-attachments/assets/9ecab539-9937-4f37-92ee-a3ead193d6ac" />

**Routing**
<img width="1852" height="903" alt="Screenshot 2025-07-29 185022" src="https://github.com/user-attachments/assets/1c33b709-93b7-4a08-81d0-c7b58e537e81" />
<img width="1853" height="505" alt="Screenshot 2025-07-30 175231" src="https://github.com/user-attachments/assets/1a0038ba-6100-4473-b3db-b90d596c6e27" />
<img width="1852" height="918" alt="routing" src="https://github.com/user-attachments/assets/26d7286a-5e80-4040-8960-0c9006bc4a71" />
<img width="1841" height="912" alt="Screenshot 2025-07-29 190229" src="https://github.com/user-attachments/assets/2e960fa2-3caf-4d39-99e6-bfb2747c5a95" />
<img width="1748" height="890" alt="Screenshot 2025-07-29 190359" src="https://github.com/user-attachments/assets/d3cc2791-c395-4fb3-bd54-f2905c7f5c99" />
<img width="1742" height="895" alt="Screenshot 2025-07-29 190416" src="https://github.com/user-attachments/assets/c8b8a5fd-7a7c-4a43-bdc7-a9378b4e07b9" />
<img width="1688" height="880" alt="Screenshot 2025-07-29 190436" src="https://github.com/user-attachments/assets/db974f00-d023-45df-acfe-24e865549b76" />
<img width="1853" height="915" alt="Screenshot 2025-07-29 194603" src="https://github.com/user-attachments/assets/c2643529-ff21-4734-a34a-dbd7a7ec5990" />

**Parasitic Extraction & Final STA**
<img width="1857" height="936" alt="Screenshot 2025-07-29 195516" src="https://github.com/user-attachments/assets/5d7777a5-7e44-4c99-a3c7-7ad856b9cfe1" />

**Final Layouts**
The final DEF and GDSII layouts represent the culmination of the entire physical design process.
<img width="751" height="817" alt="Screenshot 2025-07-30 133612" src="https://github.com/user-attachments/assets/09ab689e-e161-4e35-a7cf-3023380250a4" />
<img width="1055" height="857" alt="Screenshot 2025-07-29 224044" src="https://github.com/user-attachments/assets/ace98a9a-4742-4409-8a97-660841e75a81" /># Digital-VLSI-SoC-Openlane-sky130A

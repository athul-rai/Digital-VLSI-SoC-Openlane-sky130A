## Physical Design using OpenLANE & Sky130 PDK

## 📘 Project Introduction
This project documents the complete physical design (PD) flow for a `picorv32a` RISC-V core, from RTL to a final GDSII layout. It leverages the fully automated, open-source **OpenLANE** framework and the **Skywater 130nm (Sky130) PDK**. This work was completed as part of the Digital VLSI SoC Design and Planning Workshop by VLSI System Design (VSD) Corp, providing a practical, hands-on journey through the entire chip implementation process.

OpenLANE orchestrates a suite of powerful EDA tools—including Yosys, OpenROAD, Magic, and Netgen—to automate the entire flow, from logic synthesis to final layout generation, enabling the efficient creation of hard macros and full chip designs.




## 📁 Repository Structure

- [DAY 1](#DAY-1)
  - [Theory](#DAY-1)
    - [Introduction to RISC-V](#Introduction-to-RISC-V)
    - [Simplified RTL to GDSII Flow](#Simplified-RTL-to-GDSII-Flow)
    - [OpenLane Flow](#OpenLane-Flow)
  - [Lab](#Lab-1)
    - [Synthesis](#Lab-1)
    - [Estimation of Flip Flop Ratio](#Estimation-of-Flip-Flop-Ratio)
    - [Slack](#Slack)
      
- [DAY 2](#Day-2)
  - [Theory](#DAY-2)
    - [Floorplan](#Floorplan)
    - [Placement](#Placement)
  - [Lab](#Lab-2)
    - [Floorplan](#Floor-Plan)
    - [Placement](#Place-Ment)
    - [Characterization](#Library-Characterization)
    - [Estimation of area of the die](#Estimation-of-area-of-the-die)
      
- [Day 3](#DAY-3)
  - [Theory](#DAY-3)
    - [Designing a Library Cell](#Designing-a-Library-Cell)
    - [steps for simulation in ngspice](#Follow-these-steps-for-simulation-in-ngspice)
    - [SPICE Switching Threshold and Propagation Delay](#SPICE-Switching-Threshold-and-Propagation-Delay)
    - [16-Mask CMOS Process](#16-Mask-CMOS-Process)
  - [Lab](#Lab-3)
    - [Spice extraction of inverter in magic](#Spice-extraction-of-inverter-in-magic)
    - [Post-Layout Spice simulation (ngspice)](#Post-Layout-Spice-simulation-(ngspice))
    - [Fix Tech File DRC via Magic](#Fix-Tech-File-DRC-via-Magic)
      
- [DAY 4](#DAY-4)
  - [Theory](#DAY-4)
    - [Delay Table](#Delay-Table)
    - [Timing Analysis (using Ideal Clocks)](#Timing-Analysis-(using-Ideal-Clocks))
    - [Clock Tree Synthesis Stage](#Clock-Tree-Synthesis-Stage)
  - [Lab](#Lab-4)
    
- [DAY 5](#DAY-5)
  - [Lab](#Lab-5)
  


# DAY-1
# Inception-of-Open-source-EDA,OpenLane-and-Sky130-PDK

we explore the fundamental physical elements of an integrated circuit (IC) : 

**Package, Chip, Pads, Core, Die, and IPs** : 
- The QFN-48 (Quad Flat No-lead) package is a surface-mounted IC package with 48 pins used to connect the chip to the outside world.

- Inside the package is the die, the actual silicon chip where transistors are fabricated.

- The core is the functional part of the die where the logic (e.g., processor, memory) is implemented.

- Surrounding the core are pads, which act as electrical contact points for interfacing signals from the core to the pins on the package.
  
- The chip as a whole includes the die, package, and connections.

The core of the chip will contain two types of blocks:

**Foundry IP Blocks** (e.g. ADC, DAC, PLL, and SRAM) = blocks which requires some amount of intelligent techniques to build which can only be designed by foundries.

**Macro blocks** (e.g. RISC-V SOC and SPI) = pure digital logic blocks compared to IP's which might require some analog parts.

<img width="848" height="853" alt="182751377-2810d388-21b0-4df1-b1d4-c72176d80d28" src="https://github.com/user-attachments/assets/d6c9f443-dc08-4587-972a-c46c9b927dad" />

## Introduction to RISC-V

- RISC-V stands for Reduced Instruction Set Computing – Version 5 and is designed with simplicity and modularity in mind.

- Unlike proprietary ISAs like ARM or x86, RISC-V is open, meaning anyone can implement or modify it freely.

- The architecture consists of a base integer set and optional extensions (e.g., for multiplication, atomic operations, floating point).

- RISC-V is ideal for education, research, and industrial design due to its transparency and scalability.

## Simplified RTL to GDSII Flow 

*Synthesis*

The Register Transfer Level (RTL) code, typically written in Verilog or VHDL, is translated into a gate-level netlist. This netlist is composed of logic gates and components from a pre-defined standard cell library. These cells have fixed sizes and electrical characteristics, provided by the Process Design Kit (PDK). During synthesis, optimizations such as constant propagation, logic simplification, and technology mapping are performed to ensure an area-efficient and logically equivalent design.

*Floor Planning and Power Planning*

This step involves defining the physical layout of the chip, including the die area, core area, margins, and reserved regions for IP blocks or macros. Floorplanning also includes decisions on I/O pin placement and defining power domains.
In power planning, a Power Distribution Network (PDN) is created to deliver clean power across the chip. The power rails and straps are usually placed on upper metal layers since they are thicker and have lower resistance, which helps reduce IR drop and ensures reliable power delivery.

*Placement*

Placement determines the exact physical location of standard cells within the defined floorplan. This process occurs in two phases:

- Global Placement: Estimates optimal positions to reduce wirelength and congestion, but may not obey all design rules.

- Detailed Placement: Adjusts the global result to ensure legal placement while minimizing additional wirelength or timing penalties.

*Clock Tree Synthesis (CTS)*

A clock tree is built to distribute the clock signal to all sequential elements (flip-flops) in the design. Since all flip-flops must receive the clock signal simultaneously to avoid timing violations like skew and jitter, structures such as H-trees or X-trees are used. CTS also buffers the clock signal and ensures balanced timing paths across different regions.

*Routing*

Routing connects the logically associated pins (nets) across the placed cells using horizontal and vertical metal layers defined in the PDK.

- Global Routing identifies approximate routing paths.

- Detailed Routing determines exact wiring with specific tracks, vias, and metal layers.
 
- The Sky130 PDK provides six metal layers, each with defined widths, spacings, pitches, and via rules to guide the router for legal and             manufacturable connections.

*Verification Before Sign-off*

Before the final design is ready for fabrication, several verification steps are required:

- DRC (Design Rule Check): Ensures the physical layout complies with all foundry-imposed rules like minimum spacing, width, enclosure, etc.

- LVS (Layout Versus Schematic): Compares the layout netlist against the schematic/netlist from the synthesis phase to confirm logical equivalence.

- Timing Analysis: Static Timing Analysis (STA) verifies that all setup and hold time constraints are satisfied across all paths and corners         (process, voltage, temperature).
The final Result is [GDSII file format.](https://anysilicon.com/semipedia/gdsii/)

RTL to GDSII flow : [Openlane](https://efabless.com/openlane)

## OpenLane Flow

<img width="848" height="=500" alt="182759711-6b9352ec-7652-4589-af31-53a409eb2830" src="https://github.com/user-attachments/assets/f3cdde11-73e1-496e-9486-3d02f62ff65d"/>


OpenLane is an automated RTL to GDSII flow that utilizes various components such as OpenROAD, Yosys, Magic, Netgen, and custom scripts for design exploration and optimization.

The flow performs all ASIC implementation steps from RTL down to GDSII. OpenLane Interactive Mode Commands include a variety of functions such as setting the current netlist, running logic verification, setting the current def file, preparing lef files, preparing a liberty file, generating an exclude list file, sourcing configurations, specifying a path to save config_in.tcl, preparing a run, specifying a design folder, overwriting an existing run, specifying a path to save the run, specifying a name for a specific run, creating a tcl configuration file for a design, setting the verilog source code file, specifying the design's configuration file, setting a verbose output level, generating a padframe, saving views of a given run, changing save paths for various file types, labeling pins of a macro def, generating a verilog netlist from a def file, creating an obstruction, setting tracks on a layer, extracting core dimensions, running SPEF extraction and Static Timing Analysis, running antenna checks, saving environment variables, running OpenSTA timing analysis, checking for unmapped cells, checking for assign statements, and checking if the LEF was properly read.

- *Antenna Rules Violation* : long wire segments will act as antennna and will accumulate charges, this might damage the connected transistor gates. Solution is to either use bridging or antenna diode insertion to leak away the charges.  
### OpenLane Directory Hierarchy:

 
├── OOpenLane             -> directory where the tool can be invoked (run docker first)
│   ├── designs          -> All designs must be extracted from this folder
│   │   │   ├── picorv32a -> Design used as case study for this workshop
│   |   |   ├── ...
|   |   ├── ...
├── pdks                 -> contains pdk related files 
│   ├── skywater-pdk     -> all Skywater 130nm PDKs
│   ├── open-pdks        -> contains scripts that makes the commerical PDK (which is normally just compatible to commercial tools) to also be compatible with the open-source EDA tool
│   ├── sky130A          -> pdk variant made especially compatible for open-source tools
│   │   │  ├── libs.ref  -> files specific to node process (timing lib, cell lef, tech lef) for example is `sky130_fd_sc_hd` (Sky130nm Foundry Standard Cell High Density)  
│   │   │  ├── libs.tech -> files specific for the tool (klayout,netgen,magic...) 


Inside a specific design folder contains a config.tcl which overrides the default settings on OpenLANE. These configurations are specific to a design (e.g. clock period, clock port, verilog files...). The priority order for the OpenLANE settings:
1. sky130_xxxxx_config.tcl in OpenLane/designs/[design]/
2. config.tcl in OpenLane/designs/[design]/
3. Default values in OpenLane/configuration/



### Lab [Day 1] - Determine Flip-flop Ratio:
The task is to find the flip-flop ratio ratio for the design picorv32a. This is the ratio of the number of flip flops to the total number of cells. For the OpenLane installation, the steps are very straight forward and can be found on the [OpenLane repo](https://github.com/The-OpenROAD-Project/OpenLane).

*1. Run OpenLANE:*
 - $ make mount = Open the docker platform inside the openlane/
 - % flow.tcl -interactive = run script for automating the whole RTL to GDSII flow but in step by step -interactive mode
 - % package require openlane 0.9 == retrives all dependencies for running v0.9 of OpenLANE  
 
 ![GDS Layout](https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day1/Screenshot%202025-07-31%20123740.png)


 
*2. Design Setup Stage:*
 - % prep -design picorv32a = Setup the filesystem where the OpenLANE tools can dump the outputs. This also creates a run/ folder inside the specific design directory which contains the command log files, results, and the reports dump by each tools. These folders will be empty for now except for lef files generated by this design setup stage. This merged the [cell LEF files](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) .lef and [technology LEF files](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) .tlef generating merged.nom.lef inside run/tmp/
 

![GDS Layout 2](https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day1/Screenshot%202025-07-31%20123906.png)

*3. Run synthesis:*
 - % run_synthesis = Run yosys RTL synthesis, ABC scripts (for technology mapping), and OpenSTA.  
 
![Flip-Flop Ratio](https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day1/flip-flop-ratio.png)

*The flipflop ratio is (number of flip flops)/(total number of cells) is 1613/14876 = 0.10843. Or 10.843%*

#### Slack 

refers to the difference between the required arrival time and the actual arrival time of a signal. If the signal arrives earlier than required, it's called positive slack, indicating the design meets timing. If it arrives later, it's negative slack, meaning a timing violation has occurred. Positive slack is desirable, while negative slack must be fixed to ensure reliable circuit operation.

In Order to fix negetive slack we change the clock period to ```55.00``` in ```sky130_xxxxx_config.tcl``` 

```before```

<img width="848" height="255" alt="Screenshot 2025-07-24 123323" src="https://github.com/user-attachments/assets/9fe5c38e-a80a-4875-9cfc-4d0b81cf11c3" />


```After```

<img width="848" height="255" alt="Screenshot 2025-07-24 131231" src="https://github.com/user-attachments/assets/ae72bbe7-8a79-40b1-80b4-6af999df93fb" />

# Day 2 
##The following topics are addresed:

Floorplanning
- Calculating utilization factor (area occupied by netlist / total area of core) and aspect ratio of the core (height/width)
- Defining pre-placed cells location, which are placed before standard cells
- Usage of decoupling capacitor to counter fluctuation of power delivered to cells and noise, which can cause undefined logic values and unstable circuit functioning
- Power planning to provide power to cells

Placement
- Bind netlist to physical cells and determine size. On library, one can find same cells of different sizes depending on the needs of the circuit
- Place the netlist on the core, respecting preplaced cells and finding the best locations to guarantee signal integrity and physical proximity to where the signals should travel
- Place repeaters to strenghten signals of cells that are distant

Cell design flow
- Standard cells are kept at a library, such as AND, inverter, buffer, etc
- For inputs of the flow: PDKs, DRC and LVS rules, SPICE models, library and uuser defined specs
- It has some design steps: circuit design, layout and characterization
- Then, for output, we get circuit description language (CDL) files, GDSII, LEF, extracted SPICE netlist (.cir), timing, noise, power libs and function
- For the characterization flow, the cell's behavior should be analyzed under various conditions as to create implementation models

Timing characterization
- One should note the various timing threshold definitions, such as the slew, how much time the cell takes to respond to an input and the corresponding outputs
- The propagation delay is also crucial to compare how the output in a cell behaves to an input, as the different delays in communicating cells can cause unexpected results if not properly addressed
- Transition time points at the slew time of inputs and outputs, calculated using the threshold values
- The waveforms are a way for visualization of this step


## Lab 2 

#### Floor Plan

- Run Floorplan : ```run floor_plan```

- To view our floorplan in Magic we need :

  1. Magic technology file (sky130A.tech)
  2. Def file of floorplan
  3. Merged LEF file

  head over to the following directory to view the results of floorplan using Magic :

  ```cd /Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<date>/results/floorplan```

  To invoke magic use the command :

  ```magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &```
  
  <img src="https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day2/floorplan.png" width="800" alt="Floorplan"/>


  *Result* : 

  <img width="848" height="732" alt="Screenshot 2025-07-24 133352" src="https://github.com/user-attachments/assets/b7d0b4e7-4386-4b35-90fe-e37f92b2afa1" />

  To center the view, press "s" to select whole die then press "v" to center the view. Point the cursor to a cell then press "s" to select it, zoom into it by pressing 'z". Type "what" in tkcon to display          information of selected object. These objects might be IO pin, decap cell, or well taps as shown below.

  <img width="848" height="907" alt="1.png" src="https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day2/1.png" />
  
  if we zoom, we can see that some of the micro, IO pad, and tap-cells have been placed appropriately.

  <img width="848" height="668" alt="2.png" src="https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day2/2.png" />
  
  To get information about the selected object press ```s``` and type ```what``` in console - same as in the above image

#### Place Ment 

- To do a placement in OpenLane : ```run_placement```

- Viewing Placement in Magic :

  ```magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &```

  <img width="848" height="42" alt="placement" src="https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day2/placement.png" />

  *results* :

  <img width="848" height="728" alt="3.png" src="https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day2/3.png" />

  <img width="848" height="545" alt="4.png" src="https://github.com/user-attachments/assets/dc7bd71a-8625-4cd5-abad-92c53610c52d" />
  
  If we zoom, we the core with all the standard cells placed in between power can ground rail

# Day 3
# Design a Library Cell using Magic Layout and Ngspice Characterization

# CMOS Inverter Fabrication Process

This document outlines the process flow for fabricating a CMOS inverter using standard bulk silicon technology.

---

## 1. Substrate Selection
- **Material**: P-type silicon wafer (⟨100⟩ orientation)
- **Resistivity**: 5–50 Ω·cm
- **Key Note**: Substrate doping must be **less** than well doping.

---

## 2. Active Region Isolation (LOCOS)
- **Mask 1**: Photoresist to define active areas
- **Layers**:
  - Si₃N₄ (80 nm): Prevents oxide growth over active regions
  - Field Oxide (LOCOS): ~1 µm thick in exposed regions
- **Outcome**: Isolated active regions for NMOS and PMOS

---

## 3. Well Formation
- **N-Well (for PMOS)**:
  - **Mask 2**: Protects NMOS areas
  - **Implant**: Phosphorus @ 400 keV

- **P-Well (for NMOS)**:
  - **Mask 3**: Protects PMOS areas
  - **Implant**: Boron @ 200 keV
  - **Drive-in**: Thermal diffusion to deepen wells

---

## 4. Gate Formation
- **Threshold Voltage Adjustments**:
  - **Mask 4**: Boron implant (for NMOS)
  - **Mask 5**: Arsenic implant (for PMOS)

- **Gate Oxide**:
  - High-quality 10 nm SiO₂ (etch and regrow)
  
- **Polysilicon Gate**:
  - **Mask 6**: Pattern poly-Si gates

---

## 5. Lightly Doped Drain (LDD)
- **Purpose**: Reduce hot-carrier & short-channel effects
- **Implants**:
  - **Mask 7**: N⁻ (Phosphorus) for NMOS
  - **Mask 8**: P⁻ (Boron) for PMOS
- **Sidewall Spacers**:
  - SiO₂ deposition + anisotropic etch

---

## 6. Source/Drain Formation
- **Mask 9**: N⁺ implant (Arsenic) for NMOS
- **Mask 10**: P⁺ implant (Boron) for PMOS
- **Note**: Screen oxide used to prevent ion channeling

---

## 7. Contacts & Local Interconnects
- **TiN/TiSi₂ Formation**:
  - Ti sputtering → RTA @ 600–700°C
  - TiSi₂ for gate/source/drain
  - TiN for local interconnects

- **Mask 11**: Etch TiN for contacts

---

## 8. Metallization (Al/W)
- **Planarization**:
  - PSG deposition + CMP

- **Contact Holes and Vias**:
  - **Mask 12/14**: Via etching
  - **Mask 13/15**: Metal (Al or W) deposition

- **Mask 16**: Pad opening for top-level contacts

---

## 🔍 Key Insights
- PMOS width is 2–3× larger than NMOS for current balancing
- LDD reduces leakage and improves reliability
- Sidewall spacers protect LDD during S/D implantation
- TiSi₂ lowers gate resistance; TiN supports routing
- Total Masks: **16** (wells, gates, implants, contacts, metals)

---

Final Result : 

<img width="848" height="400" alt="Screenshot 2025-07-26 174207" src="https://github.com/user-attachments/assets/fa5ac8b5-675a-4151-a230-072c64d6137e" />


## Lab 3 

#### Spice extraction of inverter in magic 

- Clone vsdstdcelldesign. Copy the techfile ```sky130A.tech``` from ```pdks/sky130A/libs.tech/magic/``` to directory of the cloned repo. 
  <img width="848" height="52" alt="11.png" src="https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day3/11.png" />
  <img width="848" height="168" alt="12.png" src="https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day3/12.png" />

- View the mag file using magic ```magic -T sky130A.tech sky130_inv.mag &```
  <img width="848" height="57" alt="13.png" src="https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day3/13.png" />
  
- Make an extract file ```.ext``` by typing extract all in the tkon terminal of magic. 

- Extract the ```.spice``` file from this ext file by typing ```ext2spice cthresh 0 rthresh 0``` then ```ext2spice``` in the tkon terminal.

  <img width="848" height="260" alt="Screenshot 2025-07-26 174858" src="https://github.com/user-attachments/assets/76fb85c3-e007-4032-9925-8800b238c3e0" />

  <img width="848" height="721" alt="Screenshot 2025-07-26 172928" src="https://github.com/user-attachments/assets/9d946e89-dce5-46dd-be89-e8f93b824935" />

  
#### Post-Layout Spice simulation (ngspice)

- Open the spice file by typing ```ngspice sky130A_inv.spice``` and Generate a graph using plot y vs time a
  
  <img width="848" height="875" alt="Screenshot 2025-07-26 181256" src="https://github.com/user-attachments/assets/6b11a52d-e7dd-4626-882e-e8d24f331810" />

  <img width="848" height="487" alt="Screenshot 2025-07-31 160022" src="https://raw.githubusercontent.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/main/Day3/Screenshot%202025-07-31%20160022.png" />


#### Fix Tech File DRC via Magic


All technology-specific information comes from a technology file. This file includes such information as layer types used, electrical connectivity between types,     design rules, rules for mask generation, and rules for extracting netlists for circuit simulation - [DRC rules for SKY130nm PDK](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root)

Clone custom inverter standard cell design :

```
# Change directory to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Clone the repository with custom inverter design
git clone https://github.com/nickson-jose/vsdstdcelldesign

# Change into repository directory
cd vsdstdcelldesign
 
# Copy magic tech file to the repo directory for easy access
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .

# Check contents whether everything is present
ls

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &
```

- Inside the drc_tests/ are the .mag layout files and the sky130A.tech.
- Open magic with ```poly.mag``` as input: magic poly.mag from file --> open
- Focus on Incorrect poly.9.the spacing between polyresistor with poly or diff/tap must at least be 0.480um. Using command box in console, we can see that   the distance violated but there is no DRC violations shown. Our goal is to fix the tech file to include that DRC.
<img width="848" height="63" alt="Screenshot 2025-07-26 232040" src="https://github.com/user-attachments/assets/102cbe43-321e-451d-8e0e-82f87013bd23" />


<img width="848" height="472" alt="Screenshot 2025-07-26 221828" src="https://github.com/user-attachments/assets/48b3df34-f795-4d5b-9c47-09462b66b709" />
<img width="848" height="532" alt="Screenshot 2025-07-26 223639" src="https://github.com/user-attachments/assets/d9732a2e-a38e-49a6-9374-ab0a0183dd06" />
<img width="400" height="137" alt="Screenshot 2025-07-26 232204" src="https://github.com/user-attachments/assets/d39c2834-fb34-45d8-8f2f-93e4ffb25187" />

The current sky130A.tech file only includes spacing rules for:

- n-poly resistor to n-diffusion

- p-poly resistor to p-diffusion

We are now adding two new critical spacing rules (highlighted in green):

- Left rule: Spacing between n-poly resistor and regular poly (non-resistor)

- Right rule: Spacing between p-poly resistor and regular poly (non-resistor)

These additions ensure proper isolation between resistor and non-resistor poly layers in the design

<img width="400" height="193" alt="Screenshot 2025-07-26 223705" src="https://github.com/user-attachments/assets/db2ff8f4-d82f-454b-bf15-88568d65e412" />
<img width="400" height="151" alt="Screenshot 2025-07-26 223718" src="https://github.com/user-attachments/assets/e36aa647-54e9-460a-bd63-e89fcb35c495" />


To apply the new poly resistor spacing rules:

Run ```tech load sky130A.tech``` to update the rules.

Use ```drc check``` – violations appear as white dots on poly layers.

Inspect each violation with ```drc find```

New rules cover:

n-poly ↔ poly spacing

p-poly ↔ poly spacing

<img width="400" height="87" alt="Screenshot 2025-07-26 223505" src="https://github.com/user-attachments/assets/a6ec936c-e03e-440c-9729-59cd987577ab" />

# DAY 4 
# Pre-layout Timing Analysis and Importance of Good Clock Tree 

## Theory

The following topics are addresed:
- Creation of LEF file of the previously opened inverter to be used on the PicoRV32 design
- Analysis of the grid layout in Magic
- How to make the CTS power aware? Using logic gates that can block the clock from propagating. However, the delay needs to be studied by a delay table
- FFs need a setup time depending on the combinational logic present. They are also prone to uncertainty of the clock and jittering, which should be calculated and taken into account. Based on the characteristics of the circuit, the clock needs to be calculated thinking of the uncertainty and delays
- A good strategy to route a clock tree is using the H-tree approach, as it's able to balance the clock time and is good for scaling designs
- Crosstalk of the signals should be avoided, and the clock net shielding is good to remove internal capacitances that might affect the signals
- Since there are many buffers in the clock tree, the time required for the clock to reach from a launch FF to a capture FF needs to take those middle buffer delays in consideration
- Examples of setup and hold analysis using real clocks were shown and discussed

## Lab 4

- Tracks.info used in routing stage (`/desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd`)

  <img width="848" height="496" alt="CMOS Inverter Layout" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/111.png?raw=true" />

  <img width="848" height="490" alt="CMOS Inverter Output Waveform" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/112-3.png?raw=true" />
- Type the Command in tkcon window to set grid as tracks of locali layer
   ```grid 0.46um 0.34um 0.23um 0.17um```

 <img width="848" height="608" alt="CMOS Inverter Timing Diagram" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/115.png?raw=true" />

- Two Things to verify : Pins lies on intersections and cell width is 3. We can make use of grids to identify cell width.

- Extracting `LEF` file

  - open the mag file 
    ```magic -T sky130A.tech sky130_inv.mag &```
  - save the inverter by your custom name save `sky130_vsdinv.mag`
    then type `lef write` in tkcon window. This will create files as shown below.
    
    <img width="848" height="146" alt="CMOS Inverter Netlist or Logs" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/116-7.png?raw=true" />
    
    

- Pluging-in Custom Inverter Cell into Openlane

  - Copy the LEF file `sky130_vsdinv.lef` and `sky130_fd_sc_hd__*` from `openlane/vsdstdcelldesign/libs` to `picorv32a/src` directory.


    <img width="848" height="267" alt="Final CMOS Inverter GDSII View" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/118.png?raw=true" />

 
  - add these commands into `config.tcl` in `Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a`
    ```
    set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"
    set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__fast.lib"
    set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__slow.lib"
    set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"

    set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
    ```
    
   <img width="848" height="582" alt="CMOS Inverter Schematic or Layout Result" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/119.png?raw=true" />

  - Run docker and prepare the design picorv32a. You may make use of overwrite command : `prep -design picorv32a -tag <date> -overwrite`
 
    <img width="848" height="902" alt="Final CMOS Inverter Summary or Report" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/120.png?raw=true" />

  - running synthesis ```run_synthesis```
 
    <img width="848" height="481" alt="Timing Report or STA Result" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/121.png?raw=true" />
 
    <img width="848" height="897" alt="Power Report or Analysis Result" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/122.png?raw=true" />
 
    we get to see --> chip area, wns (worst timing violation) and tns (total negative slack)

    merged.lef in tmp directory with our custom inverter as macro
    
    <img width="848" height="497" alt="Post-Layout Simulation Results" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/123.png?raw=true" />

    <img width="848" height="921" alt="Final Layout or LVS Summary" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/124.png?raw=true" />

- run floorplan

 <img width="1847" height="1200" alt="Overall Project Flow or Dashboard" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/125.png?raw=true" />

  <img width="1832" height="245" alt="OpenLane Flow Completion Banner" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/126.png?raw=true" />

- Post-Synthesis timing analysis

  Since we are having improved timing of 0 wn. so what we will do is we will do timing analysis on Synthesis that has lot of violations
  - we gradually improve and reduce our slack by replacing cells with better one and to deliver improved delay.
  This is **timing ECO fixes**

  First we do is running the synthesis --> include newly added lef to openlane flow --> set ::env(SYNTH_SIZING) 1 --> set                ::env(SYNTH_MAX_FANOUT) 4 --> run_synthesis

 <img width="1832" height="245" alt="Final Report or Summary Chart" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/127.png?raw=true" />
  

  Later in `cd Desktop/work/tools/openlane_working_dir/openlane` we write command `sta pre_sta.conf`

  *sta pre_sta.conf*
  <img width="1838" height="901" alt="Final OpenLane Run Summary or Analysis" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/128.png?raw=true" />

  *my_base.sdc*
  <img width="1832" height="891" alt="OpenLane Final Output Summary or Metrics" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/129.png?raw=true" />

  <img width="1433" height="756" alt="OpenLane Design Completion Logs or Report" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/130.png?raw=true" />
  
  <img width="1817" height="908" alt="Final OpenLane Run Output or Summary Table" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/131.png?raw=true" />

  Replacing some cells to reduce slack

  <img width="1855" height="437" alt="OpenLane Report Summary or Final Output" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/132.png?raw=true" />

  report_net -connections _11672_ [we see the driver pins] 
  <img width="1857" height="910" alt="OpenLane Timing Analysis Report" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/133.png?raw=true" />

  replace_cell _14510_ sky130_fd_sc_hd__or3_4 [we replace some cells with better suited ones for driving 4 fanouts]
  <img width="1847" height="921" alt="OpenLane Power Analysis Report" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/134.png?raw=true" />

  To check the report and see the changes in slack we use command : report_checks -fields {net cap slew input_pins} -digits 4

  Slack reduced

 <img width="1841" height="905" alt="OpenLane Power Report Details" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/135.png?raw=true" />

  report_checks -from _29052_ -to _30440_ -through _14510_
  <img width="1842" height="912" alt="OpenLane Detailed Report" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/136.png?raw=true" />

  <img width="1852" height="916" alt="OpenLane Power and Timing Summary" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/137.png?raw=true" />

  report_net -connections _11668_
  
  replace_cell _14506_ sky130_fd_sc_hd__or4_4

  <img width="1855" height="645" alt="OpenLane Power Consumption Graph" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/138.png?raw=true" />

  Slack Reduced
  <img width="1853" height="880" alt="OpenLane Timing and Power Analysis" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/139.png?raw=true" />

  Intially : -23.90
  
  now : -22.9860
- Run CTS
- 
  - run synthesis,floorplan,placement and then run cts (`run_cts`)

  <img width="1297" height="457" alt="OpenLane Flow Completion Screenshot" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/140.png?raw=true" />

  <img width="1847" height="272" alt="OpenLane Run Summary Banner" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/141.png?raw=true" />
  

- Post-CTS Timing analysis : OpenROAD

  command : `openroad`

  <img width="1257" height="480" alt="OpenLane Detailed Run Output" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/142.png?raw=true" />

 <img width="1841" height="620" alt="OpenLane Final Analysis Report" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/143.png?raw=true" />
  
  removing 'sky130_fd_sc_hd__clkbuf_1 and running CTS 

 <img width="1518" height="655" alt="OpenLane Detailed Analysis" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/144.png?raw=true" />
  
  echo $::env(CTS_CLK_BUFFER_LIST) [Checking current value of CTS_CLK_BUFFER_LIST]

  <img width="1518" height="655" alt="OpenLane Detailed Analysis 2" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/145.png?raw=true" />

  running openROAD command again
  
  ```
  openroad

  read_lef /openLANE_flow/designs/picorv32a/runs/<date>/tmp/merged.lef

  read_def /openLANE_flow/designs/picorv32a/runs/<date>/results/cts/picorv32a.cts.def

  write_db pico_cts1.db

  Loading the created database in OpenROAD
  read_db pico_cts.db

  read_verilog /openLANE_flow/designs/picorv32a/runs/<date>/results/synthesis/picorv32a.synthesis_cts.v

  read_liberty $::env(LIB_SYNTH_COMPLETE)

  link_design picorv32a

  read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

  set_propagated_clock [all_clocks]

  report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

  report_clock_skew -hold

  report_clock_skew -setup
  ```
  <img width="1848" height="916" alt="OpenLane Final Detailed Report" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/146.png?raw=true" />

  <img width="1337" height="740" alt="OpenLane Run Results Overview" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/147.png?raw=true" />

  Checking clock skew for setup and hold
  <img width="1852" height="945" alt="OpenLane Timing and Power Report" src="https://github.com/athul-rai/Digital-VLSI-SoC-Openlane-sky130A/blob/main/Day4/148.png?raw=true" />



# DAY 5 
# Final steps for RTL2GDS using tritonRoute and openSTA

## Lab 5 

- Performing (PDN) Power Distribution Network

  - In docker open openlane andd perform prepare the design and include newly added `lef` to openlane flow.
  - set SYNTH_STRATEGY --> DELAY 3
  - set SYNTH_SIZING --> 1
  - run synthesis, floorplan, placement and finally cts
  - after cts is done we do power distribution network PDN
  - gen_pdn

  <img width="1845" height="898" alt="Screenshot 2025-07-29 183947" src="https://github.com/user-attachments/assets/0623ac22-6ec7-4ea5-9456-c67c1de65eab" />

  <img width="1852" height="906" alt="Screenshot 2025-07-29 184133" src="https://github.com/user-attachments/assets/944c3ba9-5daf-4bd0-8a20-96b47da0d1c6" />

  <img width="1810" height="887" alt="Screenshot 2025-07-29 184343" src="https://github.com/user-attachments/assets/23dc0a50-d4e4-4b89-8e1e-8fab538c8f30" />

  <img width="1847" height="902" alt="Screenshot 2025-07-29 184540" src="https://github.com/user-attachments/assets/f948ace1-3fe0-488a-9a35-a88aca1b8b36" />

  <img width="1856" height="898" alt="Screenshot 2025-07-29 184602" src="https://github.com/user-attachments/assets/4d1d219b-4dc2-4011-9633-dfe76a9c24be" />

  - In `<date>/tmp/floorplan/` directory we  have the pdn
    to open we use command : `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef      def read 14-pdn.def &`

  <img width="1606" height="818" alt="Screenshot 2025-07-29 184757" src="https://github.com/user-attachments/assets/d80cc966-ecb3-4adc-a923-1d4fe65a3cc7" />

  <img width="1502" height="838" alt="Screenshot 2025-07-29 184832" src="https://github.com/user-attachments/assets/8979c7cf-74b0-4249-b8ba-4a8595d78207" />

  <img width="1506" height="837" alt="Screenshot 2025-07-29 184913" src="https://github.com/user-attachments/assets/9ecab539-9937-4f37-92ee-a3ead193d6ac" />

- Perfrom detailed routing using TritonRoute

  - run routing using command --> `run_routing`
 
    <img width="1852" height="903" alt="Screenshot 2025-07-29 185022" src="https://github.com/user-attachments/assets/1c33b709-93b7-4a08-81d0-c7b58e537e81" />

    <img width="1853" height="505" alt="Screenshot 2025-07-30 175231" src="https://github.com/user-attachments/assets/1a0038ba-6100-4473-b3db-b90d596c6e27" />

    <img width="1852" height="918" alt="routing" src="https://github.com/user-attachments/assets/26d7286a-5e80-4040-8960-0c9006bc4a71" />
  - go to `<date>/results/routing/` directory
 
    load the def file : `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def      read picorv32a.def &`

    <img width="1841" height="912" alt="Screenshot 2025-07-29 190229" src="https://github.com/user-attachments/assets/2e960fa2-3caf-4d39-99e6-bfb2747c5a95" />

    <img width="1748" height="890" alt="Screenshot 2025-07-29 190359" src="https://github.com/user-attachments/assets/d3cc2791-c395-4fb3-bd54-f2905c7f5c99" />

    <img width="1742" height="895" alt="Screenshot 2025-07-29 190416" src="https://github.com/user-attachments/assets/c8b8a5fd-7a7c-4a43-bdc7-a9378b4e07b9" />

    <img width="1688" height="880" alt="Screenshot 2025-07-29 190436" src="https://github.com/user-attachments/assets/db974f00-d023-45df-acfe-24e865549b76" />

    fast route guide inside `<date>/tmp/routing`

    <img width="1853" height="915" alt="Screenshot 2025-07-29 194603" src="https://github.com/user-attachments/assets/c2643529-ff21-4734-a34a-dbd7a7ec5990" />

- Post-Route parasitic extraction using SPEF extractor
  
  for the extracting, we use [SPEF_EXTRACTOR](https://github.com/HanyMoussa/SPEF_EXTRACTOR)
  
  ```
  cd Desktop/work/tools/SPEF_EXTRACTOR
  
  python3 main.py /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<date>/tmp/merged.lef      /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<date>/results/routing/picorv32a.def
  ```
  
- Post-Route OpenSTA timing analysis

  <img width="1857" height="936" alt="Screenshot 2025-07-29 195516" src="https://github.com/user-attachments/assets/5d7777a5-7e44-4c99-a3c7-7ad856b9cfe1" />
  
  

**Picorv32a.def.png**


<img width="751" height="817" alt="Screenshot 2025-07-30 133612" src="https://github.com/user-attachments/assets/09ab689e-e161-4e35-a7cf-3023380250a4" />


**Picorv32a.gds.png**


<img width="1055" height="857" alt="Screenshot 2025-07-29 224044" src="https://github.com/user-attachments/assets/ace98a9a-4742-4409-8a97-660841e75a81" />



**GDS**

Stands for Graphic Design System. This is the file that is sent to the foundry and is called "tape-out".

In openLane use the command run_magic

The GDSII file is generated in the `results/magic` directory


**[DEF](https://teamvlsi.com/2020/08/def-file-in-vlsi-design-exchange.html)**

**[GDSII](https://en.wikipedia.org/wiki/GDSII)**


# Acknowledgements

[Kunal Ghosh](https://github.com/kunalg123) – Founder, VLSI System Design Corp.

[Nickson Jose](https://github.com/nickson-jose)– Developer & Contributor, Open Source Physical Design


     




    






  
  
  

  

  

    

 
    
    

    

    
 
    







  

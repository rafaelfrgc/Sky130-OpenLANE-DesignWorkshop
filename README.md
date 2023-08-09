# Sky130-OpenLANE-DesignWorkshop
This is a documentation of the core concepts of what was presented in the 5-day workshop  "Advanced Physical Design using OpenLANE/Sky130" by VLSI System Design (VSD), where a project with complete RTL to GDSII flow for PicoRV32a SoC is executed with Openlane using Skywater130nm PDK. Custom designed standard cells with Sky130 PDK are also used in the flow. Timing Optimisations are carried out. Slack violations are removed. DRC is verified.

# Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK
## How to talk to computers 

This section is an introduction to computers and chips with related terms and concepts. It covers:
 1. Introduction to what is meant by a "chip", covering what are packages, like the QFN-48 Package, and the parts of a package like pads, core, die, IPs and foundries.  
 2. Introduction to RISC-V:  
 This is an brief introduction to the contents of the course, talking about how a programming language is translated to hardware operations through compilation, HDLs, and layout of the chip, especiffically using the RISC-V ISA.
 3. From Software Applications to Hardware
 This topic talks about the process to execute a software application in a hardware chip, and the steps involved in the process, like the compiler, assembler and the OS. It also describes what is an ISA, the RTL description of the chip using a HDL, the netlist synthesis and the physical design of the netlist.

 ## SoC design and OpenLANE

This section talks about the Open Source ASIC Flow in general.

### History and components of ASIC design
The components needed to characterize a complete ASIC, like Register Transfer Level (RTL) design tools, Electronic Design Automation (EDA), and Process Design Kits (PDK), being the PDK the latest one to be open-sourced, by the Skywater 130 nm process.

### Simplified RTL2GDS Flow

The workflow and steps needed to make a an ASIC, from RTL to GDSII, which is the final format prior to manufacturing. The steps are: Synthesis, Floorplanning and Powerplanning, Placement, Clock Tree Synthesis, Routing, Sign-off and GDSII Generation, in this order. The data from the PDK is used in this flow.

- Synthesis: Conversion of RTL from HDL to circuit netlist of logic gates using the standard cell library
- Floor & Power Planning: Floorplanning is the process of partitioning and placing blocks (like SRAM, IO, etc) and macros in the chip die. Power planning is the process of distributing power and ground across the chip.
- Placement: Placement is the process of placing the gate level netlist standard cells in the chip die aligned in the rows. It is divided in two steps:
  - Global Placement: Optimal position of the cells in the die, may overlap
  - Detailed Placement: Minimal positioning
- Clock Tree Synthesis (CTS): The clock tree is the distribution of the clock signal to all the sequential elements in the chip. It is divided in two steps:
  - Clock Tree Synthesis: The clock tree is built
  - Clock Tree Routing: The clock tree is routed
- Routing: The interconnections between the standard cells are made using the metal layers of the PDK (metal 1, metal 2, etc) and the vias (vertical interconnect access) between them. It is divided in two steps:
  - Global Routing: The approximate route is made using routing guides
  - Detailed Routing: The exact route is made 
- Signoff: The final verification of the design is made in steps:
  - Physical Verification: Design Rule Check (DRC) and Layout vs Schematic (LVS) are made to check the design is correct and matches the netlist and the PDK rules 
  - Timing Analysis: The timing of the design is checked to ensure the design meets the timing constraints of the design (like clock frequency) through Static Timing Analysis (STA).

The Skywater 130nm PDK uses 6 metal layers to perform CTS, PDN generation, and interconnect routing.

### OpenLANE

OpenLANE is an open-source digital design flow that is used to produce clean GDSII files with no human intervention. It is based on several open-source tools, including OpenROAD, Yosys, Magic, Netgen, Fault, and custom methodology scripts for design exploration and optimization. It was created by the OpenROAD project, which is a collaborative effort to build an open-source, end-to-end, digital design flow for VLSI circuits. The term “clean” refers to the fact that the file has been checked for errors and is ready for use in manufacturing.

### OpenLANE ASIC Design Flow

The OpenLANE ASIC design flow is based on the following steps:

<!-- Add OpenLANE chart image -->

1. RTL Synthesis: The RTL code is converted to a gate-level netlist using Yosys, which is an open-source synthesis tool. Also, abc is used to perform technology mapping, which is the process of mapping the logic gates to the standard cells of the PDK.

2. STA - Static Timing Analysis: The timing of the design is checked to ensure the design meets the timing constraints of the design (like clock frequency). OpenSTA is used for this.

3. DFT - Design for Test: This is a methodology to add testability features to the hardware product design. These added features make it easier to develop and apply manufacturing tests to the designed hardware. The purpose of these tests is to validate that the product hardware contains no manufacturing defects that could adversely affect the product’s correct functioning. Fault is used for this. Fault is a tool that can be used to generate test patterns for stuck-at, transition, and path delay faults.

4. OpenROAD - Inside the OpenROAD app, many stages take place, like:
    - Floorplanning;
    - Placement: During the placement stage, OpenROAD is used to place standard cells on the floorplan. The tool uses a global placement algorithm to determine the approximate location of each cell, followed by a detailed placement algorithm to refine the cell positions and minimize wirelength;
    - Clock tree synthesis (CTS) - OpenROAD is used to construct a clock tree that distributes the clock signal to sequential elements in the design. The tool uses a clock tree synthesis algorithm to create a balanced clock tree with minimal skew;
    - Global routing - OpenROAD is used to create interconnects between the placed standard cells. The tool uses a global routing algorithm to determine the approximate routing paths for each net, followed by a detailed routing algorithm to create the final routing solution.

5. LEC - Logic Equivalence Check: This process is used to verify that the logical functionality of a design remains intact throughout the various stages of the design process, such as synthesis, place and route, sign-offs, engineering change orders (ECOs), and numerous optimizations. Yosys is used for this.

6. Detailed routing: During the detailed routing stage in the OpenLANE flow, TritonRoute takes as input a guide file generated by the global router and performs detailed routing to create interconnects between the placed standard cells. The tool uses an end-to-end detailed routing scheme that is capable of comprehending connectivity and design rule constraints to produce a DRC-clean routing solution.

7. Fake antenna diodes insertion: This process is used to prevent the chip from getting damaged due to electrostatic discharge (ESD). The diodes are inserted in the design at the places where the antenna violation is found.

8. RC extraction - is a step in the OpenLANE flow where the parasitic resistance and capacitance of the interconnects in the design are extracted. This information is used to perform accurate timing analysis and to optimize the design for performance, power, and area, using a tool such as SPEF-Extractor.

9. STA - Static Timing Analysis: The timing of the design is checked to ensure the design meets the timing constraints of the design (like clock frequency). OpenSTA is used for this.

10. Physical verification: The design is checked for errors using tools such as Magic and netgen.

11. GDSII generation: The final GDSII file is generated using Magic.

## Open-source EDA tools

This section talks about the open-source EDA tools used in the OpenLANE flow.

### OpenLANE directory structure

The OpenLANE directory structure is as follows:

- pdks
  - skywater-pdk: The Skywater 130nm PDK from the foundry;
  - open_pdks: The Open PDKs, which are responsible for the integration of the foundry PDKs with the OpenLANE flow;
  - sky130A: The OpenLANE PDK, which is the Open PDK for the Skywater 130nm PDK;
- openlane: The OpenLANE flow

### Synthesis

First, OpenLANE is started and the picorv32a design files are prepped and synthesized using the following commands:

```
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
```
After synthesis completes, a "runs" folder is generated in the picorv32a design folder, which contains the synthesis results. The log files containing the results are located in the following folder:

```picorv32a/runs/<timestamp>/logs/synthesis```

<!-- Add Synthesis folder -->

To calculate the flop ratio, 

<!-- Add Synthesis log -->

```
Flop ratio = Number of D Flip flops 
             ______________________
             Total Number of cells
```

```
dfxtp_2 = 1613,
Number of cells = 16885,
Flop ratio = 1613/16885 = 9.55%
```

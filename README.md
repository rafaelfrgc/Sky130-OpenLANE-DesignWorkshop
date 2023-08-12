# Sky130-OpenLANE-DesignWorkshop
This is a documentation of the core concepts of what was presented in the 5-day workshop  "Advanced Physical Design using OpenLANE/Sky130" by VLSI System Design (VSD), where a project with complete RTL to GDSII flow for PicoRV32a SoC is executed with Openlane using Skywater130nm PDK. Custom designed standard cells with Sky130 PDK are also used in the flow. Timing Optimisations are carried out. Slack violations are removed. DRC is verified.


## Table of Contents
1. [Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK](#day-1---inception-of-open-source-eda-openlane-and-sky130-pdk)
  - [How to Talk to computers?](#how-to-talk-to-computers?)
  - [SoC design and OpenLANE](#soc-design-and-openlane)
    - [History and components of ASIC design](#history-and-components-of-asic-design)
    - [Simplified RTL2GDS Flow](#simplified-rtl2gds-flow)
    - [OpenLANE](#openlane)
    - [OpenLANE ASIC Design Flow](#openlane-asic-design-flow)
  - [Open Source EDA Tools](#open-source-eda-tools)
    - [OpenLANE directory structure](#openlane-directory-structure)
    - [Synthesis](#synthesis)
2. [Day 2 - Good floorplan vs bad floorplan and introduction to library cells](#day-2---good-floorplan-vs-bad-floorplan-and-introduction-to-library-cells)
  - [Chip Floor planning considerations](#chip-floor-planning-considerations)
    - [Utilization Factor and Aspect Ratio](#utilization-factor-and-aspect-ratio)
    - [Pre-placed cells](#pre-placed-cells)
    - [Decoupling capacitors](#decoupling-capacitors)
    - [Power Planning](#power-planning)
    - [Pin Placement](#pin-placement)
    - [Floorplan in OpenLANE and view in Magic](#floorplan-in-openlane-and-view-in-magic)
  - [Placement in OpenLANE and view in Magic](#placement-in-openlane-and-view-in-magic)
    - [Library binding and placement](#library-binding-and-placement)
      - [Placement in OpenLANE using RePlace](#placemente-in-openlane-using-replace)
  - [Cell design and characterization flows](#cell-design-and-characterization-flows)
    - [Inputs](#inputs)
    - [Design steps](#design-steps)
      - [Circuit design](#circuit-design)
      - [Layout design](#layout-design)
      - [Characterization](#characterization)
      - [Timing threshold voltages](#timing-threshold-voltages)
      - [Propagation delay](#propagation-delay)
      - [Transition time](#transition-time)
      - [Output](#output)
3. [Day 3 - Design library cell using Magic Layout and ngspice characterization](#day-3---design-library-cell-using-magic-layout-and-ngspice-characterization)
  - [Labs for CMOS inverter ngspice simulations](#labs-for-cmos-inverter-ngspice-simulations)
    - [Spice deck](#spice-deck)
    - [CMOS inverter](#cmos-inverter)
    - [16-mask CMOS process](#16-mask-cmos-process)
    - [Layout in Magic](#layout-in-magic)
    - [SPICE Extraction](#spice-extraction)
    - [Characterization of the inverter](#characterization-of-the-inverter)
4. [Day 4 - Pre-layout timing analysis and importance of good clock tree](#day-4---pre-layout-timing-analysis-and-importance-of-good-clock-tree)
  - [Inserting custom cell design in OpenLANE](#inserting-custom-cell-design-in-openlane)
    - [Developing LEF file](#developing-lef-file)
    - [Integrating custom cell in OpenLANE](#integrating-custom-cell-in-openlane)
  - [Timing modelling using delay tables](#timing-modelling-using-delay-tables)
    - [Timing analysis and clock signal](#timing-analysis-and-clock-signal)
    - [Timing analysis using OpenSTA](#timing-analysis-using-opensta)
      - [Post-synthesis timing analysis](#post-synthesis-timing-analysis)
5. [Final steps for RTL2GDS](#final-steps-for-rtl2gds)
  - [Routing and DRC](#routing-and-drc)
  - [Power distribution network (PDN) and routing](#power-distribution-network-pdn-and-routing)
  - [SPEF Extraction and GDSII Generation](#spef-extraction-and-gdsii-generation)
6. [Acknowledgements](#acknowledgements)



# Day 1 - Inception of open-source EDA, OpenLANE and Sky130 PDK
## How to talk to computers?

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

![OpenLANE chart](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day1/Images/OpenLANEFlowChart.png)

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

To calculate the flop ratio using information from the log, 

![Synthesis log](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day1/Images/SynthesisLog.png)

```
Flop ratio = Number of D Flip flops 
             ______________________
             Total Number of cells
```

```
dfxtp_2 = 1613,
Number of cells = 16885,
Flop ratio = 1613/16885 = 10.84%
```



# Day 2 - Good floorplan vs bad floorplan and introduction to library cells

## Chip Floor planning considerations

### Utilization factor and aspect ratio


In chip floor planning, it is important to define the width and height of the core area in the floorplan. The core area is where the logic is placed on the die, while the IOs are placed in the periphery. To determine the size of the core area, it is necessary to know the sizes of the logic gates, flip flops, and other standard cells. Wires are also taken into consideration, but they are not as important.

The logical cells occupy the complete area of the core. The utilization factor is calculated by dividing the area occupied by the netlist by the total area of the core. In a practical scenario, the utilization factor is typically between 0.5 and 0.6. If the utilization factor is 1, it means that the design is very dense and there is no space to add more logical cells, which is not good for the design.

Utilization Factor =  Area occupied by netlist
                     __________________________
                        Total area of core

The aspect ratio is another important consideration in chip floor planning. It is calculated as the ratio between the width and height of the core area. If the aspect ratio is 1, it means that the core area is a square. However, depending on the design requirements, different aspect ratios may be used.

Aspect Ratio =  Height
               ________
                Width


### Preplaced cells

Pre-placed cells are complex important or critical cells in a chip design whose locations are predefined before the actual placement and routing stages. These cells are mostly related to clocks, such as clock buffers and clock muxes, as well as other cells such as RAMs and ROMs. Since these cells are placed into the core before the placement and routing stage, they are called ‘pre-placed cells’. They are usually blocks consisting of a large amount of logic, and are dividided in blocks that make certain functions, which are implemented once and used many times in the design.

### Decoupling capacitors

Decoupling capactiors decouple the logic circuit from the power supply. They are used in chip design to stabilize the voltage on the power supply plane. In any design that involves semiconductor ICs, decoupling capacitors are necessary because the voltage supplied to the components is far from ideal and tends to fluctuate due to the presence of noise. Noise margin is a measure of design margins used to guarantee that circuits operate properly under defined conditions. It is the amount of noise that a CMOS circuit could withstand without compromising the operation of the circuit1. The operation environment, power supply, electric and magnetic fields, and radiation waves are all sources of noise. Unwanted noise can also be generated by on-chip transistor switching activity.

In the context of chip design, the parasitic effects of resistance, inductance, and capacitance in interconnect wires can introduce noise into the circuit. This noise can affect the voltage levels of signals and potentially cause errors in the operation of the circuit. To prevent this, chip designers must carefully analyze and optimize the interconnect wires in their designs to minimize the effects of parasitics.

The noise margin is an important consideration in this process. It ensures that any signal which is logic ‘1’ with finite noise added to it is still recognized as logic ‘1’ and not logic ‘0’. It is basically the difference between signal value and the noise value

Therefore decoupling capacitors functions as a reservoir and acts in two ways to stabilize the voltage. When the voltage increases above the rated value, the decoupling capacitor absorbs the excessive charges. Meanwhile, when the voltage drops, the decoupling capacitor releases the charges to ensure that the supply is stable.

Decoupling capacitors are typically placed close to the power supply pins of ICs and between the pre-placed cells to completely decouple them from each-other and the power supply. They must connect directly to a low impedance ground plane in order to be effective. Short traces or vias are required for this connection to minimize additional series inductance.

### Power planning

Each block on the chip, however, cannot have its own decap unlike the pre-placed macro. So, during power planning, a power ring is designed around the core, containing both VDD and VSS rings. After the ring is placed, a power mesh is designed such that power reaches all the cells easily. The power mesh consists of horizontal and vertical lines on the chip. The objective of power planning is to meet the IR drop budget. Power planning involves calculating the number of power pins required, the number of rings and straps, and the width of rings and straps.

Ground bounce is a phenomenon associated with transistor switching where the gate voltage can appear to be less than the local ground potential, causing unstable operation of a logic gate. This can happen when there exists a small inductance in the leads connecting an IC to your board. When voltage is applied to this metal wires current starts flowing through the metal layers and there is some voltage drop due to resistance of metal wires and current. This drop is called IR drop.

### Pin placement

During pin placement, netlist defines the connectivity between logic gates, is used to determine the position of I/O pads for various pins. The space between the core and die is utilized for placing these pins. The connectivity information, which is coded in either VHDL or Verilog, is used to determine the optimal placement of the pins.

After the pins have been placed, logical placement blocking of pre-placed macros is performed. This involves differentiating the area reserved for pre-placed macros from that of the pin area. This is important because it allows designers to reserve space for other components or routing channels, and can help to improve the overall layout of the chip.

### Floorplanning in OpenLANE and view in Magic

* In the floorplanning stage, in the configuration file, there are .tcl files related to each of the steps in the flow. In these files, the values of the variables are defined. The values of these variables can be changed to change the values of the parameters in the flow. The order of importance of files is as follows:

1. ```sky130A_sky130_fd_sc_hd_config.tcl``` - System default envrionment variables
2. ```config.tcl```
3. ```floorplan.tcl```

* For Floorplan envrionment variables or switches speciffically:

1. ```FP_CORE_UTIL``` - Floorplan core utilization factor
2. ```FP_ASPECT_RATIO``` - Floorpan aspect ratio
3. ```FP_CORE_MARGIN``` - Core to die margin area
4. ```FP_IO_MODE``` - Defines pin configurations (1 = equidistant/0 = not equidistant)
5. ```FP_CORE_VMETAL``` - Vertical metal layer, which is the metal layer used for power distribution
6. ```FP_CORE_HMETAL``` - Horizontal metal layer, which is the metal layer used for ground distribution

***Usually, vertical metal layer and horizontal metal layer values will be 1 more than that specified in the files***

To run floorplan in OpenLANE, the following command is used:

```run_floorplan```

![FloorplanningRunVM](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day2/Images/FloorplanningRunVM.png)

After completing the floorplan step, a .def file will have been created in the ```results/floorplan``` directory.

For viewing the .def file, we use Magic, which is a VLSI layout tool. To view the .def file, we use the following command:

```magic -T <magic tech file> lef read <lef file> def read <def file> &```

So, in this case:

```magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.min.lef def read picorv32a.floorplan.def &```

Magic basic controls are as follows:

* To zoom in and out, use the ```z``` key and then click and drag the mouse 
* To pan, use the ```p``` key and then click and drag the mouse 
* The ```v``` key is used to select a particular layer

![MagicFloorplanTopView](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day2/Images/MagicFloorplanTopView.png)

The standard cells are not placement during this stage, but they can be seen in the black boxes in the bottom left corner. It is also possible to see the decoupling capacitors:

![MagicFloorplanDecapView](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day2/Images/MagicFloorplanDecapView.png)


## Placement in OpenLANE and view in Magic

### Library binding and placement

A library contains informations about the physical cells available for use in the design. It contains information about the height, width, and other physical characteristics of each cell. It also contains information about the logical function of each cell, which is used during the binding process. In library there may also have an option similar cells (with same functionality) but with different heights and widths, which may be chosen according to our space available.

The process of binding a netlist with physical cells involves taking the logical representation of the design, as described in the netlist, and mapping it to the physical cells available in the library. This is typically done during the placement stage of the physical design flow. During this stage, the placement tool takes the netlist as input and uses the information contained in the library to select the appropriate physical cells for each logic gate or other component in the design. The tool then places these physical cells on the chip, optimizing their placement based on estimated wire length, signal integrity, and other factors. In the case that the distances between the cells are too large, the tool may also insert buffer cells to reduce the wire length, this is done to reduce the delay and preserve signal integrity.

#### Placement in OpenLANE using RePlace

In OpenLANE, the placement tool used is RePLace. To run placement in OpenLANE, the following command is used:

```run_placement```

In this stage, global placement is performed, which is a rough placement of the cells optmizing for wire length. The output of this stage is a .def file, which can be viewed in Magic, using the following command:

```magic -T <magic tech file> lef read <lef file> def read <def file> &```

Which, in this case, is:

```magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.min.lef def read picorv32a.placement.def &```


In magic,

![GlobalPlacementTopView](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day2/Images/GlobalPlacementTopView.png)

Zooming in,

![GlobalPlacementZoomedView](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day2/Images/GlobalPlacementZoomedView.png)

Note that power distribution network generation (PDN) is usually a part of the floorplan step. However, in the OpenLANE flow, floorplan does not generate PDN. The steps are - floorplan, placement CTS and then PDN.

## Cell design and characterization flows

Standard cells are pre-designed and pre-characterized digital and analog circuit components that designers can use to create their designs. These cells are the building blocks of system-on-chip (SoC) designs and include logic gates, input/output (I/O) cells, memory compilers, and other components.

A standard cell library is a collection of these standard cells, along with other files required by the place-and-route (PnR) tool for automatic placement and routing. The library contains cells with multi-drive strength and multi-threshold voltage, as well as physical-only cells. The standard cells in the library are well-defined, pre-characterized, and free from any design rule check (DRC) violations. 

### Inputs

To start a cell design flow, PDKs, DRC and LVS rules and SPICE models  given from the foundry are needed. Also, library and user-defined specs are taken into account.

### Design steps

#### Circuit design

The circuit design step that comes before layout design is typically schematic capture. Schematic capture is the process of creating a schematic diagram that represents the circuit’s connectivity and functionality. During this step, designers use a schematic capture tool to create a visual representation of the circuit, using symbols to represent components and wires to represent connections between them. Once the schematic has been completed and validated, the design can move on to the layout stage, where the physical placement of components and routing of interconnect wires is determined

#### Layout design

In the context of layout design in chip design flow, NMOS and PMOS graphs are used to represent the connectivity between transistors in a circuit. These graphs are created by representing each transistor as a vertex and connecting vertices with edges to represent the connections between transistors. NMOS and PMOS graphs are used to analyze the connectivity of the circuit and to determine an efficient placement of transistors that minimizes wire length and routing congestion.

Euler’s path is a concept from graph theory that refers to a path through a graph that visits every edge exactly once. In the context of layout design, Euler’s path can be used to determine an efficient placement of transistors on the chip. By finding an Euler path through the NMOS and PMOS graphs, designers can determine an ordering of transistors that minimizes wire length and routing congestion. Based on Euler's path a stick diagram is a simplified layout form that contains information related to each of the process steps but does not contain the actual size of individual features. The purpose of a stick diagram is to provide the designer with a good understanding of topological constraints and to quickly test several possibilities for optimum layout without actually drawing a complete mask diagram. Stick diagrams can easily be drawn by hand and are a handy intermediate form between circuit diagrams and physical layouts since they can easily be modified and corrected. They can therefore be used to anticipate and avoid possible problems when laying out a circuit.

Following, transistor sizing is another important step in chip design. It involves analyzing the circuit to determine the drive strength required for each transistor and selecting the appropriate size from the available options in the standard cell library. Transistor sizing can affect the performance and reliability of a chip, so it is important to carefully analyze and optimize transistor sizes during chip design.

After transistor sizing, placement, and routing are completed, design rule checking (DRC) is performed to ensure that no errors occurred during these steps. DRC verifies that all design rules have been followed and that there are no violations related to spacing, connections, vias, etc. Also, a SPICE netlist is extracted from the layout containing information about the parasitic capacitance, resistance and other parameters of the model. This netlist is used in the next step of the design flow, characterization.

#### Characterization

A typical standard cell characterization flow includes the following steps:

1. Read in the models and tech files: The first step is to read in the SPICE technology models and other relevant technology files. These files contain information about the process technology and device models that are used in the simulation.
2. Read extracted spice netlist: The next step is to read in the extracted SPICE netlist of the cell design. This netlist represents the circuit as a set of interconnected devices and is used as input to the simulation.
3. Recognize behavior of the cell: The characterization software analyzes the information to recognize the cell's function. This involves identifying the inputs, outputs, and internal logic of the cell.
4. Read the subcircuits: The software reads in any subcircuits used in the design. Subcircuits are pre-designed circuit blocks that can be reused in multiple designs.
5. Attach power sources: Power sources are attached to the circuit to enable simulation. This involves connecting voltage sources to the appropriate nodes in the circuit.
6. Apply stimulus to characterization setup: The software generates and applies appropriate stimulus to determine characteristics such as delay and transition time. This involves applying input signals to the circuit and simulating its response.
7. Provide necessary output capacitance loads: Output capacitance loads are provided as necessary for simulation. These loads represent the capacitance seen by the outputs of the cell and can affect its performance.
8. Provide necessary simulation commands: The final step is to provide any necessary simulation commands to control the simulation process. These commands can include options for controlling simulation accuracy, convergence, and other parameters.

Once these steps have been completed, GUNA software generates timing, noise, and power models based on the results of the simulation. These models can then be used in chip implementation flows to ensure that the final chip meets its performance, power, and area goals.

##### Timing threshold voltages

The following variables are threshold voltages used in the timing characterization of a standard cell, such as a buffer containing two inverters back-to-back. They are used to define the voltage levels at which a signal is considered to have transitioned from one logic state to another. These thresholds are typically defined as a percentage of the power supply voltage.

- `slew_low_rise_thr` (20% Typically) and `slew_high_rise_thr` (80% Typically) are used to measure the rise time (or slew rate) of a rising signal. The rise time is measured between the `slew_low_rise_thr` and `slew_high_rise_thr` voltage levels.
- `slew_low_fall_thr` (20% Typically) and `slew_high_fall_thr` (80% Typically) are used to measure the fall time (or slew rate) of a falling signal. The fall time is measured between the `slew_low_fall_thr` and `slew_high_fall_thr` voltage levels.
- `in_rise_thr` (50% Typically) and `in_fall_thr` (50% Typically) are used to measure the delay of a rising or falling input signal, respectively. The delay is measured from the point at which the input signal crosses the `in_rise_thr` or `in_fall_thr` voltage level.
- `out_rise_thr` (50% Typically) and `out_fall_thr` (50% Typically) are used to measure the delay of a rising or falling output signal, respectively. The delay is measured from the point at which the output signal crosses the `out_rise_thr` or `out_fall_thr` voltage level.

##### Propagation delay 

The propagation delay is the time it takes for a signal to travel from the input of a standard cell to its output. It is characterized by measuring the delay of the rising and falling output signals with respect to the rising and falling input signals. The delay is measured from the point at which the input signal crosses the in_rise_thr or in_fall_thr voltage level to the point at which the output signal crosses the out_rise_thr or out_fall_thr voltage level.

Having a negative delay is not physically possible and indicates an error in the characterization or simulation process. This could happen due to incorrect voltage thresholds being used, incorrect simulation settings, or errors in the circuit model. It is important to define the correct voltage thresholds for measuring delays and slew rates, as these values are used in static timing analysis to ensure that the circuit meets its timing requirements.

##### Transition time

The transition time, also known as the slew rate, is the time it takes for a signal to transition from one logic state to another. It is characterized by measuring the rise time and fall time of the signal. The rise time is measured between the slew_low_rise_thr and slew_high_rise_thr voltage levels, while the fall time is measured between the slew_low_fall_thr and slew_high_fall_thr voltage levels.

The transition time is an important parameter in digital circuit design, as it affects the maximum operating frequency of the circuit. A slower transition time can result in longer propagation delays, which can limit the maximum operating frequency of the circuit. On the other hand, a faster transition time can result in increased power consumption and electromagnetic interference.

#### Output

Typically the output of the cell design flow consists of the following files:

- CDL (Circuit Description Language): This file describes the circuit's netlist in a human-readable text format. It includes information about the circuit's components, such as transistors, resistors, and capacitors, as well as their interconnections. The CDL file is used by simulation tools to verify the behavior of the circuit.

- LEF (Library Exchange Format): This file describes the physical layout of the cell in a human-readable text format. It includes information about the cell's size, shape, and pin locations, as well as the layers and geometries used to construct the cell. The LEF file is used by place-and-route tools to integrate the cell into a larger chip design.

- GDSII (Graphic Data System II): This file describes the geometric shapes and text labels used to represent the cell's layout in a binary format. It is used by mask-making tools to create photomasks for manufacturing the cell.

- Extracted SPICE netlist (.cir): This file describes the circuit's netlist in a human-readable text format, including parasitic capacitances and resistances extracted from the layout. It is used by simulation tools to verify the behavior of the circuit, taking into account the effects of parasitics.

- Timing, noise, and power .lib files: These files describe the cell's timing, noise, and power characteristics in a human-readable text format. They include information about the cell's input and output capacitances, transition times, propagation delays, and power consumption. These files are used by static timing analysis tools to verify that the circuit meets its timing requirements and by power analysis tools to estimate its power consumption.

# Day 3 - Design of library cells using Magic Layout and ngspice characterization

Note that it is possible to make changes on the OpenLANE flows and re-run the flow to see the impact of the changes. To do that, you need to change the variables in the configuration and rerun the flow. 

## Labs for CMOS inverter ngspice simulations

### Spice deck

First, it is needed to create a SPICE deck, also known as a SPICE input file or source file, is a text-based file that describes a circuit to be simulated using the SPICE (Simulation Program with Integrated Circuit Emphasis) software. The file consists of three main parts: data statements, control statements, and output statements.

Data statements describe the components of the circuit and their interconnections. Control statements specify the type of analysis to be performed on the circuit, such as DC, AC, or transient analysis. Output statements specify what outputs are to be printed or plotted.

The term “deck” comes from the early days of SPICE when input files were printed on punch cards and fed into a computer as a deck of cards.

### CMOS inverter

To make a CMOS inverter, it is needed to use the following components, first we define the connectivity between the components and then we define the parameters or values of each component. Following that, we name the nodes to characterize the positions of the componentes relative to each other

### 16-mask CMOS process

1. Substrate selection: A moderately high resistivity, (100) orientation, P-type silicon wafer is selected as the base material.

2. Wafer cleaning, thermal oxidation, nitride deposition, and photoresist spinning and baking: In this step, the wafer is first cleaned to remove any contaminants that could interfere with the subsequent processing steps. Next, a layer of silicon dioxide (SiO2) is grown on the surface of the wafer through thermal oxidation. This involves heating the wafer in an oxygen-rich environment to form a thin layer of SiO2 on its surface. After the oxide layer has been grown, a layer of silicon nitride (Si3N4) is deposited on top of it using low-pressure chemical vapor deposition (LPCVD). This layer serves as a mask for the subsequent LOCOS (Local Oxidation of Silicon) process. Finally, a layer of photoresist is spun onto the wafer and baked to prepare it for the next photolithography step.


3. Mask #1 - Active area definition: In this step, the first photolithography mask is used to define the active areas of the circuit. The mask is aligned with the wafer and exposed to ultraviolet light, which transfers the pattern from the mask onto the photoresist layer. The exposed areas of the photoresist are then developed and removed, leaving behind a patterned photoresist layer that protects the underlying nitride layer in the active areas. The nitride layer is then dry etched to remove it from the exposed areas, revealing the underlying SiO2 layer in the active areas.


4. Field oxide growth: In this step, a thick field oxide layer is grown to electrically isolate the active areas from each other. This is done using a LOCOS (Local Oxidation of Silicon) process, which involves selectively oxidizing the exposed SiO2 in the field regions while protecting the active areas with the remaining nitride layer. The result is a thick field oxide layer that surrounds and electrically isolates each active area.

5. Mask #2 - NMOS well formation: In this step, the second photolithography mask is used to form the wells for the NMOS devices. The mask is aligned with the wafer and exposed to ultraviolet light, which transfers the pattern from the mask onto the photoresist layer. The exposed areas of the photoresist are then developed and removed, leaving behind a patterned photoresist layer that protects the underlying silicon in the NMOS well regions. A boron (B+) implant is then performed to introduce boron atoms into the exposed silicon, forming P-type wells for the NMOS devices.

6. Mask #3 - PMOS well formation: In this step, the third photolithography mask is used to form the wells for the PMOS devices. The process is similar to that used for the NMOS well formation, but with a different mask pattern and a phosphorus (P+) implant instead of a boron (B+) implant. The result is N-type wells for the PMOS devices.

7. High-temperature drive-in: In this step, a high-temperature drive-in process is performed to produce the final well depths and repair any implant damage. This involves heating the wafer to a high temperature to drive the implanted boron and phosphorus atoms deeper into the silicon and activate them. The result is well-defined P-type and N-type wells for the NMOS and PMOS devices, respectively.

8. Mask #4 - NMOS threshold voltage adjustment: In this step, the fourth photolithography mask is used to adjust the threshold voltage of the NMOS devices. The mask is aligned with the wafer and exposed to ultraviolet light, which transfers the pattern from the mask onto the photoresist layer. The exposed areas of the photoresist are then developed and removed, leaving behind a patterned photoresist layer that protects the PMOS devices. A threshold voltage adjust implant is then performed on the NMOS devices to adjust their threshold voltage to the desired value.

9. Mask #5 - PMOS threshold voltage adjustment: In this step, the fifth photolithography mask is used to adjust the threshold voltage of the PMOS devices. The process is similar to that used for the NMOS threshold voltage adjustment, but with a different mask pattern and implant species. The result is PMOS devices with their threshold voltage adjusted to the desired value.

10. Gate oxide growth and polysilicon deposition: In this step, the thin oxide over the active regions is stripped and a new gate oxide layer is grown. This involves heating the wafer in an oxygen-rich environment to form a thin layer of SiO2 on its surface. After the gate oxide layer has been grown, a layer of polysilicon is deposited on top of it using low-pressure chemical vapor deposition (LPCVD). This layer will be used to form the gates of the MOS transistors.

11. Mask #6 - Polysilicon gate definition: In this step, the sixth photolithography mask is used to define the polysilicon gates of the MOS transistors. The mask is aligned with the wafer and exposed to ultraviolet light, which transfers the pattern from the mask onto the photoresist layer. The exposed areas of the photoresist are then developed and removed, leaving behind a patterned photoresist layer that protects the underlying polysilicon in the gate regions. The polysilicon layer is then etched to remove it from the exposed areas, leaving behind well-defined polysilicon gates for the MOS transistors.

12. Lightly Doped Drain (LDD) regions formation in both the NMOS and PMOS devices. The LDD structure consists of lightly-doped source/drain regions adjacent to the gate, with heavily-doped source/drain regions laterally displaced from the gate electrode.

The LDD structure is used to reduce the electric field at the drain pinchoff region, which helps to minimize two important effects: the hot electron effect and the short channel effect.

The hot electron effect occurs when electrons in the channel of a MOSFET gain enough energy to overcome the potential barrier between the silicon and the gate oxide, allowing them to be injected into the gate oxide. This can result in a permanent change in the threshold voltage of the device, reducing its reliability.

The short channel effect occurs when the channel length of a MOSFET is comparable to the depletion layer widths of the source and drain junctions. This can result in a number of issues, including drain-induced barrier lowering, velocity saturation, and reduced subthreshold slope.

To form LDD regions, additional masking steps are used after the polysilicon gate definition step. These steps involve implanting dopants into the source and drain regions with a lower dose than that used for the heavily-doped source/drain regions. The exact details of these steps may vary depending on the specific technology and design requirements. 

13. Formation of the source and drain regions of the MOSFETs. This is typically done through a process called ion implantation, where ions of a dopant material are accelerated into the silicon substrate to create regions of n+ type and p+ type material. After the implantation, a high-temperature annealing process is used to activate the dopants and repair any damage to the crystal lattice caused by the implantation. During annealing, the silicon wafer is heated to a high temperature, typically between 950°C and 1050°C, for a period of time. This allows the dopant atoms to diffuse into the silicon lattice and occupy substitutional sites, becoming electrically active. The annealing process also helps to reduce the number of defects in the crystal lattice, improving the electrical characteristics of the MOSFETs.

14. Interlayer dielectric deposition and contact hole etch are the following steps. The interlayer dielectric is a layer of insulating material that separates the metal layers in an integrated circuit. The contact hole etch is the process of creating openings in the interlayer dielectric layer to allow for electrical connections between the metal layers.

The interlayer dielectric deposition process typically involves depositing a layer of insulating material, such as silicon dioxide or a low-k dielectric material, onto the surface of the wafer. This can be done using techniques such as chemical vapor deposition (CVD) or atomic layer deposition (ALD).

After the interlayer dielectric has been deposited, the contact hole etch process can begin. This involves using a patterned mask to selectively etch away portions of the interlayer dielectric layer, creating openings or “holes” that will allow for electrical connections between the metal layers. The etching process can be done using techniques such as reactive ion etching (RIE) or deep reactive ion etching (DRIE).

15. Metal deposition and patterning is a process of creating thin metal films on a substrate using a combination of metal deposition, metal removal, and photolithography. There are two widely used methods for metal patterning: Subtractive Transfer and Additive Transfer (lift-off method).

Chemical Mechanical Planarization (CMP) is a process used to planarize or smooth the surface of the wafer. CMP is used to reduce the surface topography of the wafer, which can improve the performance of the devices being fabricated on the wafer.

After CMP, a barrier layer such as Titanium Nitride (TiN) is deposited. TiN serves as an adhesion layer for tungsten deposition on dielectric materials. Tungsten is then deposited using techniques such as Chemical Vapor Deposition (CVD).

A top Silicon Nitride (SiN) layer can be used as a capping layer after silicide formation to prevent oxidation during subsequent processing steps4. This helps to protect the chip and improve its reliability.

### Layout in Magic

We shall integrate a custom CMOS inverter layout in the picorv32a chip. The layout is cloned from the following github:

```git clone https://github.com/nickson-jose/vsdstdcelldesign```

In the vsdstdcelldesign folder we use the following command to open the layout in Magic:

```magic -T sky130A.tech sky130_inv.mag```

Note: the sky130A.tech file was extracted from the pdks folder.

The layout is shown below:

![CMOSInverterMagic](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day3/Images/CMOSInverterMagic.png)

We can also verifiy the interconnects in the layout by selecting the components with "s" three times, so, selecting the connection between the drains:

![CMOSInverterInterconnect.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day3/Images/CMOSInverterInterconnect.png)

Also, note about the LEF or library exchange format: A format that tells us about cell boundaries, VDD and GND lines. It contains no info about the logic of circuit and is also used to protect the IP


### SPICE Extraction

Now, to extract the SPICE from .mag to .spice we use the following commands:

```
extract all
ext2spice cthresh 0 rethresh 0
ext2spice

```

![InverterSPICEExtractionTkz.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day3/Images/InverterSPICEExtractionTkz.png)

Then, we modify the .spice file to make a transient analysis of the cell. The .spice file is shown below:

![SPICENetlist.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day3/Images/SPICENetlist.png)

Runnning the simulation with the command:

```ngspice sky130_inv.spice```

Then, the output "y" is to be plotted with "time" and swept over the input "a":

```plot y vs time a```

We get the following plot:

![YvsTPlot.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day3/Images/YvsTPlot.png)

Then, using the plot we can calculate the rise time, fall time and propagation delay as explained in previous sections:

Rise Time: The time taken for the output signal to go from 20% of its max value to 80% of its max value.
Fall Time: The time taken for the output signal to go from 80% of its max value to 20% of its max value.
Propagation Delay(Rising): The time difference between the points where the input and output are at 50% of their magnitude when the signal rises from 0V to max value (50% output rise - 50% input fall).
Propagation Delay(Falling): The time difference between the points where the input and output are at 50% of their magnitude when the signal falls from max value to 0V (50% output fall - 50% input rise).

### Characterization of the inverter

The values extracted from the ngspice graphs are:

1. Rise Time:

![RiseTime.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day3/Images/RiseTime.png)

Tr = 2.26331ns - 2.18416ns = 0.07915ns

2. Fall Time:

![FallTime.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day3/Images/FallTime.png)

Tf = 4.08308ns - 4.03873ns = 0.04435ns

3. Propagation Delay(Rising):

![CellRiseDelay.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day3/Images/CellRiseDelay.png)

Trd = 2.21922ns - 2.2ns = 0.01922ns

4. Propagation Delay(Falling):

![CellFallDelay.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day3/Images/CellFallDelay.png)

Tfd = 4.05819ns - 4.05ns = 0.00819ns


# Day 4 - Pre-layout timing analysis and importance of good clock tree

## Inserting custom cell design in OpenLANE

### Developing LEF file 

First, we need to guarantee that the layout of the cell in magic is correct conforming to the PnR tool, for that:

* The ports (Y and A in this case) must be on the interconnects of the grids;
* The width and height of the cell must be an odd multiple of the track pitch;

For that, it is needed to convert the grid size to the track pitch. The tracks info file contains the pitch and offset of the tracks. The pitch is the distance between the center of two adjacent tracks. The offset is the distance from the origin to the center of the first track. The tracks info file is shown below:

Now, to configure the grid in magic we use the following command:

```grid 0.46um 0.34um 0.23um 0.17um```

The grid is shown below:

![MagicGRID.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/MagicGRID.png)


Now, it is necessary to define the name of the pins (or ports) and their attributes and uses fot the PnR tool to make right connections. For that, we select the ports in magic and use the Edit > Text option.

For port A:

![APort.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/APort.png)

For port Y:

![YPort.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/YPort.png)


For port VPWR:

![VPWRPort.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/VPWRPort.png)

For port VGND:

![VGNDPort.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/VGNDPort.png)

Now, selecting the ports areas again, we must set their purposes:

For port A:

```
port class input
port use signal
```

For port Y:

```
port class output
port class signal
```

For port VPWR:

```
port class inout
port use power
```

For port VGND:

```
port class inout
port use ground
```

Now, we generate a lef file using:

```lef write```

And the purpose of these steps is to insert a custom cell in the OpenLANE flow.

### Integrating custom cell in OpenLANE

First, we copy the generated lef file to the designs/picorv32a/src directory. Then, we copy the the libraries from te vsdstdcelldesign/libs folder to the designs/picorv32a/src folder.

Note that there are 3 types of libs, slow, typical, and fast. The typical library models the behavior of the circuit under nominal voltage, and temperature (PVT)conditions. The typical library models the behavior of the circuit under nominal PVT conditions, while the slow and fast libraries model the behavior of the circuit under worst-case PVT conditions. For example, the slow library models the behavior of the circuit when the process is slow (i.e., when the transistors are slower than expected), while the fast library models the behavior of the circuit when the process is fast (i.e., when the transistors are faster than expected).

Then, we modify config.tcl to include the new cell in the synthesis with the following enviroment variables:

```
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

The config.tcl is as follows:

![ModifiedConfigTcl.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/ModifiedConfigTcl.png)


Then, we execute the synthesis, floorplan and placement steps again, while overwriting the previous results with the new cell by using the following commands:

```
prep -design picorv32a -tag RUN_2023.08.10_04.10.39 -overwrite
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs
run_synthesis
run_floorplan
run_placement
```

![FlowReRun.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/FlowReRun.png)

Now, to view the results in magic, from the folder /results/placement,

```
magic -T /home/rafael_linux/OpenLane/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.max.lef def read picorv32a.def &
```
![MagicAfterPlacement.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/MagicAfterPlacement.png)

Using the cell manager tool, we can see the new custom cell after placement:

![MagicCustomCell.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/MagicCustomCell.png)


## Timing modelling using delay tables

Delay tables are used in Static Timing Analysis (STA) to calculate the delay of a cell in a timing path. Typically, a delay table lists the amount of delay as a function of one or more variables, such as input transition time and output load capacitance. From these table entries, the STA tool calculates each cell delay. Cell delay is the time it takes for a signal to propagate through a cell. The delay tables are contained in the library of the cell. Also, an important aspecto to take note in the delay of a cell in a trace is that of output load capacitance.

Output load capacitance refers to the total capacitive load on a trace, which is defined as the sum of the input capacitance of all the other devices sharing the trace. The capacitance of the device driving the trace is not included in this calculation. So, for examploe, output load capacitance can affect the performance of a circuit in several ways. For instance, in digital circuits, the load capacitance can impact the propagation delay of the signal, which is the amount of time it takes for the signal to go from the output of one gate to the input of another. The speed of signal transmission decreases with increasing load capacitance. Or, in analogue circuits, this capacitance impacts the frequency response and bandwidth. So, as the propagataion delay increases when the output signal of a gate takes longer to rise or fall when the load capacitance is large, a slower circuit speed, a lower working frequency, and more power consumption may be the effects of this delay.


## Timing analysis and clock signal

Another important aspecto to take into account when doing timing analysis is that of clocks. In an ideal situation, the clock signal would be a perfect square wave, with a 50% duty cycle. However, in reality, the clock signal is not perfect, and it has a finite rise and fall time. This means that the clock signal does not change instantaneously, and it takes some time to change from one state to another.

Because of that, a phenomenon called clock skew happens in synchronous digital circuit systems, where the same sourced clock signal arrives at different components at different times due to gate or wire signal propagation delay. Clock skew can be positive or negative, positive skew means that the clock signal arrives later at a cell than at the clock source and, likewise, negative skew means that the clock signal arrives earlier at a device than at the clock source. Knowing that, Clock skew can be calculated by subtracting the arrival time of the clock signal at a cell from the arrival time of the clock signal at the clock source. Clock skew can be compensated for by adding delay to the clock signal at the clock source. Other factors can also influence, such as wire-interconnect length, temperature variations, variation in intermediate devices, capacitive coupling, material imperfections, and differences in input capacitance on the clock inputs of devices using the signal.

Knowing that, in real clock design, some factors can interfere with the integrity and synchronization of the signal, such as:

- Clock jitter: deviation from true periodicity of a presumably periodic signal, often in relation to a reference clock signal;

- Clock crosstalk: a phenomenon that occurs when a signal transmitted on one circuit or channel creates an undesired effect on another circuit or channel. For that, Clock net shielding is a technique used to reduce inductive ringing and improve performance in clock distribution networks. It involves the use of shield wires, which are additional ground or power wires that are inserted between the clock signal wire and other wires to reduce loop inductance and the associated inductive effects;

So, in a variable called clock uncertainty we model these factors like jitter, crosstalk, skew, and others. Being it essentialy the maximum value a clock signal can vary from its ideal value.

An algorithm used to design clock circuits to achieve zero skew, is called H-tree. It is a clock distribution topology used to achieve minimum clock skew and good robustness against variations. It is structurally symmetric and balanced, which ensures low skew across the corners of related sequential elements. The symmetrical construction of the H-tree allows for equal wire lengths, which helps to achieve perfect synchronization between clock signals before their arrival at sub-blocks or synchronous elements. However, the H-tree topology comes at the cost of large wirelength and clock power.

Also, clock buffers are added to simplify the clock tree design. They are used to generate multiple copies of a clock signal, which can then be distributed to different parts of the circuit and also can help to reduce clock skew and improve the performance of the circuit by ensuring that the clock signals arrive at their destinations at the same time.

## Timing analysis using OpenSTA

In static timing analysis, some factors of importance are:

- Setup time: It is the minimum amount of time the data signal should be held steady before the clock event so that the data are reliably sampled by the clock. A setup constraint specifies how much time is necessary for data to be available at the input of a sequential device before the clock edge that captures the data in the device.

- Hold time: It is the minimum amount of time the data signal should be held steady in a cell after the clock event so that the data are reliably sampled by the clock. A hold constraint specifies how long the data must be held at the input of a sequential device after the clock edge that captures the data in the device.

- Slack: It is used to indicate whether timing is met along a timing path, being the difference between the desired arrival time and the actual arrival time for a signal. It determines if the design is working at the desired frequency, with a positive slack meaning that the signal can get from the startpoint to the endpoint of the timing path fast enough for the circuit to operate correctly. Positive slack indicates that the design is meeting the timing and still it can be improved, while zero slack means that the design is critically working at the desired frequency. Negative slack means that the design is not meeting the timing and it needs to be improved.


### Post-synthesis timing analysis

To run the post-synthesis timing analysis, we first create a pre_sta.conf file to use the OpenSTA tool to check the slack time:

![pre_confStaFile.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/pre_confStaFile.png)

Then, we run the OpenSTA tool outside the OpenLANE environment, using the command:

```
sta pre_sta.conf
```

The output of the command is:

![OpenSTAOutput.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/OpenSTAOutput.png)

As we can see, the slack time is negative, which means that the design is not meeting the timing and it needs to be improved. For that, we can change the fanout variable using:

```
set ::env(SYNTHESIS_FANOUT) 4
```
And also change cells with high fanout.

Also, by changing the SYNTH_SIZING, SYNTH_STRATEGY, cells with high fanout and synthesis buffer variables and running synthesis again, and fine-tuning these variables, the slack time can be reduced. For example,

![SlackOptimized.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/SlackOptimized.png)

Next, we run clock tree synthesis using the command:

```
run_cts
```
With the output:

![CTS.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day4/Images/CTS.png)


Then, we check for the slack again running Post-CTS STA in OpenROAD using the commands:

```
openroad
write_db pico_cts.db
read_db pico_cts.db
read_verilog /openlane/designs/picorv32a/runs/12-08_14-08/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a
read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock (all_clocks)
report_checks -path_delay min_max -format full_clock_expanded -digits 4
```	

# Day 5 - Final steps for RTL2GDS

## Routing and DRC

One of the algorithms used for routing is know as Lee's algorithm or the maze algorithm. It is a widely used algorithm to find the shortest path between two points in a maze. It is based on breadth-first search and always gives an optimal solution if one exists.

The main steps of the algorithm are as follows:
1. **Initialization**: Select the start point and mark it with 0.
2. **Wave expansion**: Mark all unlabeled neighbors of points marked with i with i+1, where i starts from 0 and increases by 1 until the target is reached or no more points can be marked.
3. **Backtrace**: Starting from the target point, go to the next node that has a lower mark than the current node and add this node to the path until the start point is reached.
4. **Clearance**: Block the path for future wirings and delete all marks.

While Lee's algorithm guarantees an optimal solution, it can be slow and requires considerable memory. To overcome these shortcomings, more advanced algorithms such as Line search Algorithm and Steiner Algorithms have been developed.

## Power distribution network (PDN) and routing

Now, having ran the routing, we can use the following commando to run power distribution network (PDN):

```
gen_pdn
```

This phase takes the design_cts.def file as input and generates the grid and straps for the Vdd and ground signals. These are placed around the standard cells, which are designed such that their height is a multiple of the space between the Vdd and ground rails. In this design, the pitch is 2.72.

Power enters the chip through power pads, one for Vdd and one for Gnd. From these pads, power enters rings through vias. The straps are connected to these rings, with Vdd straps connected to the Vdd ring and Gnd straps connected to the Gnd ring. There are both horizontal and vertical straps.

The power is then supplied from the straps to the standard cells by connecting the straps to the rails of the standard cells. If macros are present, then the straps attach to the rings of the macros via macro pads, and the PDN for the macro is pre-done.

In this design, straps are at metal layers 4 and 5, while standard cell rails are at metal layer 1. Vias connect across layers as required.

Then, routing is the next step, as explained in the beginning sections, the TritonRoute tool is used for routing. Some of its features are:

- MILP: stands for Mixed Integer Linear Programming, which is a mathematical optimization method that can be used to solve problems with both integer and continuous variables1. TritonRoute, an initial detailed router for advanced VLSI technologies, uses a parallel MILP-based panel routing scheme for each layer, called intra-layer parallel routing;

- Honours pre-processed route guides: A detailed router needs to honor route guides, i.e., to route within route guides as much as possible;

- Assumes that each net satisfies inter guide connectivity: This means that all unconnected terminals of a net can be traced through connected route guides.

To run routing, we use the command:

```
run_routing
```
The options for routing can be set in the config.tcl file. Also, optimisations in routing can be done by specifying the routing strategy. There are 5: 0, 1, 2, 3, 4 and 14. There is a trade-off between the optimised route and the runtime for routing. For the default setting picorv32a takes approximately 30 minutes according to the current version of TritonRoute.

## SPEF Extraction and GDSII Generation

Now, it is important to do post-routing STA analysis. For that, the first goal is to extract (SPEF), which stands for Standard Parasitic Exchange Format, which is an IEEE standard for representing the parasitic data of wires in a chip in ASCII format. SPEF extraction is done outside the OpenLANE as it has no native extraction tool.

The extractefd .spef file from the lef files can be found in the routing folder, under results. Finally, by running the command:

```
run_magic
```
The final GDSII file is generated in the magic folder under results:

![picorv32a.gds.png](https://github.com/rafaelfrgc/Sky130-OpenLANE-DesignWorkshop/blob/main/Day5/Images/picorv32a.gds.png).

# Acknowledgements

- Kunal Ghosh, Co-founder (VSD Corp. Pvt. Ltd)
- Nickson Jose, Teaching Assistant (VSD Corp. Pvt. Ltd)
- The VSD-IAT Team
- The OpenROAD project




































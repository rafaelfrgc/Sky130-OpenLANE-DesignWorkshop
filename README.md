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

#### Placement in OpenLANE using RePLace

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

## Labs for CMOS inverter ngspice simulations











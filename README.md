# Physical-Design-with-OpenLANE
Physical Design using OpenLANE / Sky130 

## Contents
- 1. Introduction
- 2. RTL to GDSII Overview
- 3. The OpenLANE Flow
- 4. Day 1: Overview of the tools, Preparation and Synthesis steps
- 5. Day 2: Floor Planning and Placement

## 1. Introduction
OpenLANE is an automated RTL to GDSII flow which uses several Open-Source tools like OpenROAD, Yosys, Magic, Netgen, Fault, OpenPhySyn and a SPEF extractor to run the flow. We can use customized scripts for modelling and optimization of the steps. It is a methodology or process designed to work specifically on Google-Skywater 130nm PDKs (Open-Source PDKs) using Open-Source tools. The target of OpenLANE is to produce clean GDSII without any human intervention and is used to produce hard macros and chips.

## 2. RTL to GDSII Overview
The flow can be divided into the following steps

1. **Chip Specification** - A Design is specified to the VLSI Design engineer whose task is to model the circuit according to the design, keeping the constraints in mind.

2. **Design entry / formal verification** - RTL design and behavioral modeling are considered for design entry, and are performed using a hardware description language (HDL), generally Verilog, VHDL or System Verilog based on the requirement. EDA tools taking the HDl files as the input, perform mapping of higher-level components to the transistor level needed for physical implementation.

      RTL Stands for Register Transfer Level, which describes the circuit using combinational logic, Registers, and Modules (IP’s or Soft Macros).
      Formal verification step is to cross verify the original circuit with the RTL design before proceeding further.
      
3. **RTL Synthesis** -  The RTL description is converted into gates by a HDL compiler and then this circuit is optimized and mapped to the standard cells from the Standard Cell Library (SCL) by a design compiler. The optimized design is also called Gate-Level Netlist. This is done two steps.
      
     - **GTECH Mapping** – The HDL netlist is mapped to generic gates whiach is then logically optimized based on AIGERs and other topologies created from the generic          mapped netlist.
      
     - **Technology Mapping** – The post-optimized GTECH netlist is mapped to the standard cells described in the PDK.
     
4. **DFT Insertion** - The circuit that checks the 'Design-For-Test' is inserted in the design.
 
5. **Floor and Power Planning** - This step includes Chip-Floor Planning, Macro-Floor Planning and Power Planning.
     - **Chip-Floor Planning** - Partition of the chip die between different system building blocks is done and I/O pads are placed.
     - **Macro-Floor Planning** - Dimensions, defenitions of the rows and Pin locations are defined in this step. Macro placement and blockages are defined before              placement occurs to ensure a legalized GDS file.
     - **Power Planning** - A grid is created with Vdd and Ground lines connected to the supply pads. Power straps are added to bring power to the middle of the chip            using higher metal layers which reduces IR drop and electro-migration problem.
  
6. **Placement** - Standard Cells are placed in their rows and overlapped such that the power lines can be connected, as defined in the technology LEF file. There are two stages, Global and Detailed.
      - **Global Placement** - The optimal placing of standard cells is done based on the proximity to the pins and other cells. There can be overlapping and the cells may not be aligned to rows in this step.
      - **Detailed Placement** - The overlapping that happened in Global Placement is corrected in this step, legalizing the design according to the constraints of placement.
      
7. **Clock Tree Synthesis** - The clock distribution network which delivers the clock to all the sequential elements is generated in this step. The target of this step is to generate a network with minimum skew. There are algorithms such as H-Tree and X-Tree algoritms to design such networks.

8. **Routing** - Interconnections are made between the standard cells using the metal layers after the CTS and PDN are created. Routing is done on routing grids to avoid DRC errors. There are few algorithms such as Maze Routing - Lee's Algorithm for this step.

9. **Final Verification** - This step verifies the functionality of the design post-routing with the original design and all other checks including pre and post-routing STA checks are made before giving out for tape out.

![asic_flow](https://user-images.githubusercontent.com/44549567/106283997-9fad7d00-6268-11eb-832c-ec61eee508d9.png)

## 3. The OpenLANE Flow

The inputs for the flow are Open-Source PDK (Skywater130 PDK in this case), and the design source/verilog file, and the output is a GDSII/LEF file.

**PDK (Process Design Kit)** - The proess design kit (PDK) is the set of inputs required to model and tape out an IC. It consists of DRC and LVS rules, SPICE Models, Library files, and User-defined specifications such as height of the standard cells / width between the power lines, and drive strength of the cells.

![openlane_flow](https://user-images.githubusercontent.com/44549567/106285544-a210d680-626a-11eb-9ddb-1166d37f03b5.png)

The tools used in this flow are as follows:

1. **Synthesis and DFT:**
      - **Yosys** - Performs RTL synthesis using GTech mapping.
      - **abc** - Performs technology mapping to the standard cells described in the PDK. The abc scripts can be integrated differently to adjust synthesis techniques based on our requirements.
      - **OpenSTA** - Performs the Static Timing Analysis on the resulting netlist to generate timing reports.
      - **Fault** – Performs Scan-chain insertion to be used for testing post fabrication. It Supports **ATPG** (Automatic Test Pattern Generation), compaction, fault coverage and simulation.

2. **Floor Planning and PDN:**
      - **Init_fp** - Defines the core area for the macros, the rows that are used for placement, and the tracks that are used for routing.
      - **Ioplacer** - Places the Input and Output ports for the Macros.
      - **PDN** - Generates the Power Distribution Network
      - **Tapcell** - Inserts welltap and decap cells in the floorplan that are placed to avoid **Latch-up** phenomenon.
     
 3. **Placement:**
      - **RePLace** - Performs global placement
      - **Resizer** - Performs optional optimizations on the design
      - **OpenPhySyn** - Performs timing optimizations on the design
      - **OpenDP** - Perfroms detailed placement to legalize the globally placed components.
      
4. **CTS:**
      - **TritonCTS** - Synthesizes the clock Tree / clock distribution network
      
5. **Layout and DRC checks:**
      - **Magic** - The layout tool which helps us check the layout after each stage, edit the design, check for DRC errors, and extract SPICE netlist from the layout.
      - **Netgen** - Performs the LVS rules chekcs on the layout.
      

## 4. Day 1: Overview of the tools, Preparation and Synthesis steps

#### Skywater130 PDK files

The Skywater PDK files on which we work are in the following directory.

![3 - files in pdks folder](https://user-images.githubusercontent.com/44549567/106295804-e6a26f00-6276-11eb-82ac-f1a798cf0194.JPG)

- Skywater-pdk – Contains all the files related to the skywater130 PDK.
- Open_pdks – Contains scripts that are used to bridge the gap between closed-source and open-source PDK to EDA tool compatibility
- Sky130A – Contains the required libraries and technology files.

![4 - sky130A files](https://user-images.githubusercontent.com/44549567/106296674-fb333700-6277-11eb-806c-d4413ac580c1.JPG)
 
The PDK file which is used here for our model

![6 - the pdk on which we are working (highlited)](https://user-images.githubusercontent.com/44549567/106296871-33d31080-6278-11eb-9f80-2a17db221736.JPG)

#### Starting OpenLANE

The OpenLANE supports both completely automated and manual / interactive mode. The interactive mode can be entered by running the following command.
```
./flow.tcl -interactive
```

![10 - starting OpenLANE](https://user-images.githubusercontent.com/44549567/106297504-ca073680-6278-11eb-927f-075dceba72b6.JPG)

#### Importing Packages

Installing the required packages

![11 - installing required packages](https://user-images.githubusercontent.com/44549567/106297977-54e83100-6279-11eb-84e5-2ce35873051f.JPG)

#### Design Files

The design files of Picorv32 i.e, the input Verilog code file and SDC files are present in the src folder, and corresponding configuration files.

![12 - files in picorv32a (in designs)](https://user-images.githubusercontent.com/44549567/106298316-c0ca9980-6279-11eb-8853-025936035b85.JPG)

The config.tcl file (configuration file)

![14 - Config tcl file](https://user-images.githubusercontent.com/44549567/106298818-65e57200-627a-11eb-80fb-c3e0a2eb33d1.JPG)

The configurations in the Sky130 config file precedes in priority to the above config.tcl file

![15 - sky130_config tcl (high priority than config tcl)](https://user-images.githubusercontent.com/44549567/106299451-2d926380-627b-11eb-983e-f8ec32304ec6.JPG)

#### Preparation step

- The preparation step creates a custom folder, which gets updated with the reports, results of all the stages of RTL to GDSII flow starting with Synthesis to Routing immediately after the stage gets run in the flow. 
- Preparation step also merges the technology LEF and cell LEF information. Technology LEF information contains layer definitions and a set of restricted design rules needed for PnR flow. The cell LEF contains obstruction information of each standard cell needed to minimize DRC errors during PnR flow.

![16 - creating design file in Openlane](https://user-images.githubusercontent.com/44549567/106300965-21a7a100-627d-11eb-9f4b-61ab620f2842.JPG)

The 'runs' folder created after running 'preparation'

![18 - new 'runs' design folder in picorv32 after preparation step](https://user-images.githubusercontent.com/44549567/106312784-38ee8a80-628d-11eb-81e8-264943c6c471.JPG)

The Config.tcl file consists of varios strategies used while running different stages of the flow.

![19 - Config tcl after prep _part1](https://user-images.githubusercontent.com/44549567/106312929-6e937380-628d-11eb-9df1-d00ddc941046.JPG)

![20 - part2](https://user-images.githubusercontent.com/44549567/106312958-781cdb80-628d-11eb-8a5c-582cd716aee4.JPG)


#### Synthesis step

Running the synthesis step

![22 - Running synthsis step](https://user-images.githubusercontent.com/44549567/106313261-ebbee880-628d-11eb-9257-0a865505ea9e.JPG)

Synthesis successful

![23 - Synthesis step successful](https://user-images.githubusercontent.com/44549567/106313316-009b7c00-628e-11eb-97b2-438d4205f971.JPG)

Post-Synthesis log report

![24 - post synthesis log report](https://user-images.githubusercontent.com/44549567/106313386-1e68e100-628e-11eb-86bb-386583ed2a13.JPG)

Synthesis results added to 'runs' folder created during 'preparation' step

![26 - post synthesis results in runs folder](https://user-images.githubusercontent.com/44549567/106313546-67b93080-628e-11eb-87b8-c9e8253986ec.JPG)


## 5. Day 2: Floor Planning and Placement

Important aspects of floor planning are:
- Core Area 
- Die Area
- Aspect Ratio
- Core Utilization
- Place Macros
- Placing I/O pins
- Power distribution network

#### Aspect Ratio 
Aspect ratio describes the shape of the chip. It is the height of the core / width of the core. Square chip has an aspect ratio of 1.

#### Utilization Factor
Utilization factor tells us about the area filled on the chip. It is defined as (Area occupied by the netlist) / (Total area of the core).

#### Macros
Macros are otherwise known as 'pre-placed cells' because they are placed earlier than the other cells. They represent a part of the circuit that replicates multiple times in a design. To avoid implementation each and every time, they are implemented only once and re-used for other times. We define locations and blockages for preplaced cells to ensure no standard cells are placed where the placeplaced cells are located.

#### Decoupling Cpacitors
Decoupling capacitors are placed local to preplaced cells during Floorplanning. Voltage drops associated with interconnect wires can heavily affect our noise margin or put it into an indeterminate state. Decoupling capacitor is a capacitor connected in parallel to the macros, hence it will charge up to the power supply voltage over time and it will work as a charge reservoir when a transition is needed by the circuit, and therefore decoupling the circuit from the main supply.

#### Power Plannning
Power planning is an essential step to lower the noise in circuits attributed to voltage droop and ground bounce. Formation of coupling capacitor between interconnect wires and the substrate takesplace, and during a transition on a net, charge associated with coupling capacitors may be dumped to ground. If there are not enough ground taps, charge accumulates at the tap and the ground line will act like a large resistor, raising the ground voltage and lowering the noise margin. To bypass this issue a robust PDN with multiple power strap taps are needed to lower the resistance associated with the PDN.

#### Pin Placement
Pin placement is another important step of floorplanning to minimize timing delays, buffering and reduce power consumption. Using the connectivity information of the HDL netlist, the I/O placing is determined. Optimal pin placement results in less buffering and therefore less power consumption. After pin placement is done, we need to place logical cell blockages along the I/O ring to block the tool for placing anything in that area, as it is registered for pins.

#### Floorplan in OpenLANE

The floorplan.tcl file includes details of various parameters like which metal layer to use for which type of pin, pitch and offset distances for PDN and blockage dimensions.

![1](https://user-images.githubusercontent.com/44549567/106322200-7c4ff580-629b-11eb-8386-c5f63496cb3b.png)

Running Floorplan

![5 - running floorplan](https://user-images.githubusercontent.com/44549567/106322340-be793700-629b-11eb-822a-41dabb05589f.JPG)

The output of floorplan is a DEF file, describing the core area and standard cell placement sites.

![4](https://user-images.githubusercontent.com/44549567/106322866-8292a180-629c-11eb-9722-0942b1f87449.jpg)

The floorplan DEF file

![10 - def file after floorplan](https://user-images.githubusercontent.com/44549567/106323163-f8970880-629c-11eb-8ba2-2c88c3ac2940.JPG)


#### Floorplan in Magic

The files required to see the floorplan in magic are:
- Magic Technology file (sky130A.tech)
- Floorplan DEF file
- Merged LEF file

![11 - opening the def file in Magic](https://user-images.githubusercontent.com/44549567/106323203-064c8e00-629d-11eb-8f0b-5593a85441e2.JPG)

The layout after floorplan

![6](https://user-images.githubusercontent.com/44549567/106323493-84109980-629d-11eb-89d7-54e14e57bbdf.png)

Decap cells at the edges

![15 - endcells at the edge of the layout](https://user-images.githubusercontent.com/44549567/106323672-c89c3500-629d-11eb-9c91-99ef7aa965aa.JPG)


#### Placement

Running placement

![7](https://user-images.githubusercontent.com/44549567/106323848-19ac2900-629e-11eb-8359-0c5aecb1716f.jpg)

One of the amis of placement is convergence, acheived by getting an almost zero overflow

![8](https://user-images.githubusercontent.com/44549567/106324024-6f80d100-629e-11eb-981b-0fcdbaff084b.jpg)

Placement in Magic

![23 - opening post-placement layout in magic](https://user-images.githubusercontent.com/44549567/106324070-86272800-629e-11eb-940a-3f1735ca9c4d.JPG)

Layout after placement

![24 - post-placement layout in magic (placement of std  cells)](https://user-images.githubusercontent.com/44549567/106324113-9ccd7f00-629e-11eb-8708-e8b29691a75a.JPG)

#### Library Characterization 
The Inputs required by the characterization software are **Process Design Kits (PDKs)** which contain DRC, LVS Rules, Spice Models, Library and User-Defined Specifications such as the Width between the power lines (the cell height), drive strength etc. The Design step involves 'Layout design' and 'Characterization Flow'. The GUNA software takes the inputs and gives GDS2, LEF and extracted Spice netlist as Outputs. The Characterization is classified into Timing, Noise and Power Characterization.

#### Characterization Flow Steps 
- CMOS Spice model file with basic parameters
- Extract the Spice Netlist
- Analyze the Circuit behaviour
- Read the Sub-circuit 
- Read the Power Supply
- Apply proper Stimulus
- Provide necessary output capacitances (C_load)
- Provide necessary simulation commands


## 6. Day 3: Designing a Library cell

The advantage of OpenLANE is that we can make changes to the internal switches while running the flow. By this, we can experiment and explore the RTL to GDSII flow steps.

#### SPICE Simulations

SPICE deck contains the details of:
- Model
- Connectivity of the components
- Output load capacitance and Resistances
- Component values
- Names of the nodes
- Simulation commands

We model a CMOS inverter in Ngspice by modifying it's spice deck, analyze it's characteristics, generate required files so that we can include it in the netlist to be placed on the chip by running the flow.

#### 16 Mask CMOS process 
- **Selecting a Substrate :** A Substrate is selected to act as base layer over which other layers are formed.
- **Creating Active regions for transistors :** Silicon Dioxide (SiO2) and Silicon Nitride (si3N4) are deposited, pockets are created for P and N regions using Lithography.
- **N-Well and P-Well formation :** Ion implantation is done using Boron for P-well and Phosphorus for N-Well. Drive in diffusion is done by placing in high temperature furnace.
- **Formation of Gate :** Depending on the requirements of NA (Doping Concentration) and Cox (OXide Capacitance), lithography and ion implantation are done.
- **Lightly Doped Drain (LDD) Formation :** LDD regions are formed to avoid Hot electron effect and Short channel effect. Plasma anisotropic etching is used to leave small amount of oxide on side walls of Gate
- **Source-Drain Formation :** Very thin layer of oxide is formed over the top to prevent chanelling effect and, Source and Drain are formed using lithography and ion implantation.
- **Forming Contacts and interconnects :** Titanium is sputtered on the wafer surface and heated to form TiSi2 and TiN (2 types of metal contacts). Etching done by RCA Cleaning.
- **Higher Level Metal formation :** Planarizing the top surface, depositing upper metal layers.

#### Magic layout of Inverter standard cell

Opening the layout (mag file) in Magic 

![2 - Opening the inverter layout in Magic (mag file)](https://user-images.githubusercontent.com/44549567/106355801-4dc82e00-6320-11eb-8b7e-dd8c84e188ff.JPG)

![3 - Inverter layout in Magic](https://user-images.githubusercontent.com/44549567/106355829-7b14dc00-6320-11eb-9f1d-0f2f19d382d1.JPG)

SPICE extraction

![5 - step1 of extracting spice netlist](https://user-images.githubusercontent.com/44549567/106356028-4a35a680-6322-11eb-8664-9ac40641881d.JPG)

Extracting parasitics information

![6 - step2 of spice and parasitics extraction](https://user-images.githubusercontent.com/44549567/106356040-751ffa80-6322-11eb-9d3a-17cf4ad23f82.JPG)

Extracted SPICE file

![7 - extracted spice file](https://user-images.githubusercontent.com/44549567/106356046-8406ad00-6322-11eb-83a4-867dda42dbb0.JPG)

Modifications to Spice file for Transient analysis

![8 - modifications to spice for transient analysis](https://user-images.githubusercontent.com/44549567/106356105-042d1280-6323-11eb-981e-0ef31d2ffe33.JPG)

Running the spice file in Ngspice

![9 - opening spice file in Ngspice (last command)](https://user-images.githubusercontent.com/44549567/106356117-1eff8700-6323-11eb-9e62-e04ae49a6bb3.JPG)

Commands to plot the output vs time

![10 - plot y vs time a](https://user-images.githubusercontent.com/44549567/106356513-1492bc80-6326-11eb-91b3-0908209605ca.JPG)

Plotting the inverter output vs time and input

![11 - Inverter output vs time](https://user-images.githubusercontent.com/44549567/106356206-db594d00-6323-11eb-8288-49746cc02fcf.JPG)

#### Timing Characterization

- **Rise Delay :** Time taken for waveform to rise from Slew Low Rise Threshold (20%) to Slew High Rise Threshold (80%) of VDD.
- **Fall Delay :** Time taken for waveform to fall from Slew High Fall Threshold (80%) to Slew Low Fall Threshold (20%) of VDD.
- **Propagation Delay :** Measured between 50% of Input transition to 50% of Output transition.

#### Rise Delay (20% and 80% values)

![12 - 20% and 80% value co-ordinates of output  waveform](https://user-images.githubusercontent.com/44549567/106356470-d1384e00-6325-11eb-8be5-53e85d5bffc3.JPG)

#### Propagation Delay (50% values)

![13 - 50% value of input and output (propagation delay)](https://user-images.githubusercontent.com/44549567/106356485-eb722c00-6325-11eb-88a9-018e0e64a59f.JPG)


## Day 4: 














  
            









     

# Physical-Design-with-OpenLANE
Physical Design using OpenLANE / Sky130 

## Contents
- 1. Introduction
- 2. Running Pin Placement of picoRV32, understanding Floor Plan and Library Cells using the EDA tools
- 3. Design and Characterization of cells using Magic Layout tool and ngspice
- 4. Timing Analysis, Clock Tree Synthesis and Signal Integrity
- 5. Final steps for RTL2GDS

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









     

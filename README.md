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
 
 5. **Floor and Power Planning** - 



     

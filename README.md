# VSD-IAT: Sign-off Timing Analysis - Basics to Advanced
![vsd_iat_logo](https://user-images.githubusercontent.com/73732594/152016610-be3ef4c8-601c-40e7-af85-91dc3ae9b2a4.png)

Static Timing Analysis (STA) is a method of verifying the timing performance of a digital circuit design by analyzing the delays of the circuit's logic paths. It is typically used to ensure that the circuit will meet its timing requirements, such as setup time, hold time, and clock-to-output delays. 

It starts with basics of Static Timing Analysis, timing paths, startpoint, endpoint and combinational logic definitions. It explains setup and hold checks, how STA tools calculate setup and hold violations. Then it slowly builds up to cover all aspects of STA like multiple types of timing paths, design rule checks, checks on async pins and clock gates. After that we go into slightly advanced topics like Time borrowing on latches, timing arcs, cell delays and models, impact of clock network on STA. Since STA and timing constraints go hand in hand the workshop covers basics of all the timing constraints that an engineer should know for STA like clock definitions, clock groups, clock characteristics, port delays and timing exceptions. 

This repository contains all the details of the hands on labs done by me during 5-days Cloud Based Sign-off Timing Analysis workshop organized by VSD and vlsideepdive. The full STA flow was implemented using OpenSTA with SKY130nm PDK.

## *Contents*
------------
* [Day-1 Summary](#day-1-summary)
	
* [Day-1 Labs](#day-1-labs)
  * [OpenSTA Introduction and its basics](#opensta-introduction-and-its-basics)
  * [Input files to OpenSTA](#input-files-to-opensta)
  * [Constraints creation](#constraints-creation)
  * [OpenSTA Run script](#opensta-run-script) 
  
* [Day-2 Summary](#day-2-summary)
  
* [Day-2 Labs](#day-2-labs)
  * [Liberty File and Understanding .lib file ](#liberty-file-and-understanding-lib-file)
  * [Understanding Lib Parsing](#understanding-lib-parsing)
  * [Understanding SPEF file and SPEF parsing](#understanding-spef-file-and-spef-parsing)
  * [Understanding Timing Reports](#understanding-timing-reports)
* [Day-3 Summary](#day-3-summary)
 
* [Day-3 Labs](#day-3-labs)
  * [Understanding Slack computation](#understanding-slack-computation)
  
* [Day-4 Summary](#day-4-summary)
  
* [Day-4 Labs](#day-4-labs)
  * [Understanding clock gating check](#understanding-clock-gating-check)
  * [Understanding Async pin checks](#understanding-async-pin-checks)
  
* [Day-5 Summary](#day-5-summary)

* [Day-5 Labs](#day-5-labs)
  * [Revisit slack computation](#revisit-slack-computation)
  * [Clock Reconvergence Pessimism (CRP) Basics](#clock-reconvergence-pessimism-crp-basics)
  * [Clock Reconvergence Pessimism Removal (CRPR)](#clock-reconvergence-pessimism-removal-crpr)
  * [Engineering Change Order (ECO)](#engineering-change-order-eco)
* [Acknowledgements:](#acknowledgements)
* [Author:](#author)

--------

# Day-1 Summary

[Day-1 Summary](https://github.com/Sumanyu-Singh/VSD_IAT_Sign-off_Timing_Analysis-Basics_to_Advanced/blob/main/Theory.md#day-1-summary)

# Day-1 Labs

## OpenSTA Introduction and its basics

OpenSTA is an open-source software package for performing performance testing and analysis of integrated circuits (ICs) or electronic designs. It is designed to analyze the timing characteristics of digital circuits and provide an understanding of their performance.

OpenSTA stands for "Open System Timing Analyzer," and it is used to verify and optimize the timing of digital circuits in order to ensure that they meet the required performance specifications. It is a free tool that can be used to simulate the timing behavior of a digital circuit, which is an important step in the design and optimization process. For more details about commands related to OpenSTA, please refer [this](https://github.com/The-OpenROAD-Project/OpenSTA/blob/master/doc/OpenSTA.pdf)

An STA tool takes design, standard cell, constraints as input and perform timing checks on the design. OpenSTA works on industry formats (e.g., .v, .spef, .lib, .sdc) and is designed to be parallel and portable.

## Input files to OpenSTA

At first, we need to run below command for cloning:

     git clone https://github.com/vikkisachdeva/openSTA_sta_workshop

Also we need to install OpenSTA separately
       
The inputs to OpenSTA are design, standard cells associated with the netlist and the constraints.
Below is the tool flow for starting navigation through directories:

![flow](https://user-images.githubusercontent.com/100671647/220419564-56b35da8-e773-45e1-a628-4d93ba179072.png)

Below is the verilog file **simple.v** opened in vim editor:

![simple_v](https://user-images.githubusercontent.com/100671647/220421706-af7b19b3-0a97-4bb5-a9b4-607862a38947.png)

Standard cells information is present in .lib file as can be seen from below:
":syn off" command in vim editor turns syntax(highliting) off and ":set nu" command do line numbering

![lib](https://user-images.githubusercontent.com/100671647/220422878-01bed6a9-8973-4e8e-a769-e69aeee33788.png)

Example of nand2_1 cell from .lib file is shown below:

![nand2_1](https://user-images.githubusercontent.com/100671647/220423953-bb2cba78-823f-46c4-a368-d0454266cdeb.png)

## Constraints creation

The SDC file provided for the lab. This consists of the clock period, IO delays, input transition and capacitance delays. 
Below is the standard Synopsys Design Constraint (SDC) file (**simple.sdc**):

![sdc](https://user-images.githubusercontent.com/100671647/220425299-229a4c13-9963-48c3-8d1f-4012410dc4f6.png)

Primary ports are defined with delays with associated clock: tau2015_clk

Below are the commands for IO Delays:

	set_input_delay 5 –max –rise [get_ports inp1] –clock tau2015_clk 
	set_output_delay -10 –min –fall [get_ports out] –clock tau2015_clk
	
Command for Input transitions by environmental factors: 

    set_input_transition 10 –min –rise [get_ports inp1]
    
Command for setting Capacitive load on output pin:

    set_load –pin_load 4 [get_ports out]
    
"**report_checks**" command is used to report timing on the design.

## OpenSTA Run script

Below are the commands used for running the STA flow:

![run_script](https://user-images.githubusercontent.com/100671647/220608639-f2a4676a-72dd-4378-bd4b-57512c49febe.png)

For running the OpenSTA, we use below command

	sta run.tcl -exit | tee run1.log

Below are the reports (can be seen in "run1.log" file) after running OpenSTA:

![violated](https://user-images.githubusercontent.com/100671647/220624097-71471f6e-b7c0-4c71-aa0c-1619ed7625f6.png)
![met](https://user-images.githubusercontent.com/100671647/220625551-81839db1-2ed4-454e-bcea-b5447e3093ec.png)

In 1st, the timing checks aren't met(VIOLATED) since the slack is negative but in 2nd it is MET (slack is positive).

# Day-2 Summary

[Day-2 Summary](https://github.com/Sumanyu-Singh/VSD_IAT_Sign-off_Timing_Analysis-Basics_to_Advanced/blob/main/Theory.md#day-2-summary)

# Day-2 Labs

## Liberty File and Understanding .lib file
A "liberty file" is a file format that provides timing and power information for cells in a standard cell library. A standard cell library is a collection of pre-designed cells, such as gates and flip-flops, that can be used to create digital circuits.

The liberty file contains timing information about the delay of each cell and its input and output pins. This information is critical for accurate timing analysis of a circuit. It also provides information on the power consumption of each cell, which is important for power analysis and optimization.

The .lib file is an ASCII representation of the timing and powerparameters associated with any cell in a particular semiconductor technology The .lib file contains timing models and data to calculate I/O delay paths, Timing check values and Interconnect delays.

**Below is picture showing descriptions of liberty file**

![understanding_lib_file](https://user-images.githubusercontent.com/100671647/220627166-6a01e5f4-3076-46ef-9bd0-39db740039b1.png)

**Exercises_2.1:**

**Q1) Find all the cells in simple_max.lib.**

- To find number of cells in simple_max.lib file type below command: 
```
    more simple_max.lib | grep -c " End cell"
```
- In order to list all cells in a library and dump in a "all_cell" file, type below command:
```
    more simple_max.lib | grep " End cell" | tee all_cell
```
**Snapshot below showing execution of above commands** (a lot more cells are there in the list which are not shown)

![cell_findings](https://user-images.githubusercontent.com/100671647/220631367-43db3baa-4deb-444a-8755-97e5cce7f33d.png)


**Q2) Find all the pins of the cell NAND2_X1 in simple_max.lib**
- From below picture we can see that there are 1 output and 2 input pins for cell NAND2_x1

![pins_of_cell](https://user-images.githubusercontent.com/100671647/220633263-2fdb469a-2c6e-4b83-868d-67e3ff03a159.png)


**Q3) What difference you see between NAND2_X1 and NAND3_X1**
- NAND2_X1 is 2 input nand gate and NAND3_X1 is 3 input nand gate.
- Also Capacitance values are differnet.

![image](https://user-images.githubusercontent.com/100671647/220634764-4e9b0de2-9cc9-4517-ba00-8e0eda4d0f9d.png)

The max capacitance increases multifold, and the number of input pins.

**Q4) What is the difference between ‘simple_max.lib’ and ‘simple_min.lib’**
- I compared the file and found they have differnet values in cell_ fall, fall_transition, cell_rise and rise_transitionof all the cells.
- Fabrication process variations could either increase or decrease the delay of a cell. So we need to set early and late value while setting the derate factor. STA tool would consider early or late timing derate based on the path and type of analysis.
By using below command I checked the difference between both files:
    
```
    gvimdiff simple_max.lib simple_min.lib
```
      
![diff](https://user-images.githubusercontent.com/100671647/220635945-26cfcc2d-2a79-4f22-a50c-81ef3e5864bc.png)

## Understanding Lib Parsing

Below is the **run.tcl** file for lab2:

![run_tcl](https://user-images.githubusercontent.com/100671647/220642949-602644d2-b249-4d60-85d4-edae21355312.png)

Below is **simple.sdc** file for lab2:

![simple_sdc](https://user-images.githubusercontent.com/100671647/220643625-bd1a1312-4d07-4d3a-af8e-36aa7a88193d.png)

## Understanding SPEF file and SPEF parsing

In static timing analysis (STA), a "spef" file (also known as a Standard Parasitic Exchange Format file) is a commonly used file format that describes the parasitic capacitance and resistance of a digital circuit.

During STA, a spef file is used to model the interconnect delays and timing constraints of a design. The file contains information about the routing topology of the design, including the location and size of interconnect wires, the capacitance and resistance of the wires, and the location and size of the devices (gates and pins) in the design.

By analyzing the spef file, STA tools can estimate the timing delays of the interconnect wires and the devices, and determine if the design meets its timing requirements. The spef file is typically generated by layout tools during the physical design stage of a chip.

SPEF can be generated by place-and-route tool or a parasitic extraction tool, and then this SPEF is used by timing analysis tool for checking the timing, in-circuit simulation or to perform crosstalk analysis.

Below is the **simple.spef** file:

![image](https://user-images.githubusercontent.com/100671647/220648831-07e7ee4d-4732-4800-8f80-5ec2ac19911f.png)


Parasitics can be represented at many different levels.
A Typical SPEF File has 4 main sections

1. Header
2. Name Map
3. Top Level Ports
4. Parasitic description

Below is the command for running STA tool
```
    sta run.tcl -exit | tee run_2.log
```

![spef_run_sta](https://user-images.githubusercontent.com/100671647/220645368-e43dec30-db90-4c5b-b2d3-27c644955ef7.png)

## Understanding Timing Reports 

Below is the timing report we have obtained from the netlist and other inputs being parsed into the OpenSTA tool.

![report](https://user-images.githubusercontent.com/100671647/220647268-42ead6c6-9e18-40c1-a8c4-b79b133222a5.png)

**By Changing number of paths to 5 we see below reports getting generated and saved in run_3.log file**

![image](https://user-images.githubusercontent.com/100671647/220650837-cec789c9-c71d-4d4f-a0b7-f561230066e8.png)

Report in run3.log is below:

![image](https://user-images.githubusercontent.com/100671647/220656752-c8800963-d67b-4dcc-a485-7267efc7d5d5.png)

# Day-3 Summary

[Day-3 Summary](https://github.com/Sumanyu-Singh/VSD_IAT_Sign-off_Timing_Analysis-Basics_to_Advanced/blob/main/Theory.md#day-3-summary)

# Day-3 Labs

## Understanding Slack computation

Cosider below example circuit for slack calculatio for different paths:

![ckt_example](https://user-images.githubusercontent.com/100671647/220704024-88b79a19-e358-49e4-987f-c1a2c9dfca36.png)

Below is **s27.v** file:

![image](https://user-images.githubusercontent.com/100671647/220706153-5129bba8-a5f7-4710-8701-54044cc101c4.png)

Below is **s27.sdc** file:

![image](https://user-images.githubusercontent.com/100671647/220706540-f2b80f4b-d92d-448f-9400-64c4e374074e.png)

For running STA use below command:
```
    sta run.tcl -exit | tee run_1.log
```
**Below is the run.tcl content and run_1.log file generated after STA run**:

![image](https://user-images.githubusercontent.com/100671647/220708715-f35bca8e-3e6c-4900-85ce-9148e76208a5.png)

![image](https://user-images.githubusercontent.com/100671647/220709336-839ee7c3-7738-418a-9d3e-371f6753d55f.png)

**To get a detailed report consisting caps delay, net delays, slew rate input pins delays use below command:**
```
      report_checks -path_delay min_max -fields {nets cap slew input_pins} -digits {4}
```
![run_tcl](https://user-images.githubusercontent.com/100671647/220710442-39a32777-f313-473d-bc1e-ad3c7b80d35a.png)

**Generated run_1.log file is as below:**

![run1_log](https://user-images.githubusercontent.com/100671647/220711265-c0c1e634-0b76-492d-93b8-48a370096c72.png)

Add below command in **run.tcl**, this will report slack compulation w.r.t 2 path as shown in below image 
```
    report_checks –from F1/CK -endpoint_count 100
```
Below is generated result of STA run(run_1.log file):

![image](https://user-images.githubusercontent.com/100671647/220716644-244a83fd-b283-4c20-95d6-505a8b847779.png)

**Exercises:**

- **Q1) Change the number of paths being reported to 100**
Add below command in **run.tcl** to do so, This will report slack compulation w.r.t 8 path as shown in below image (generated result) which is the maximum number pf possible path in the given design :-

```
    report_checks –from F1/CK -endpoint_count 100
```

**Then, Run STA with below command:**
```
   sta run.tcl -exit | tee out.txt
```
**Generated Result of above command, shows slack computation of all possible paths for given design example**


# Day-4 Summary

[Day-4 Summary](https://github.com/Sumanyu-Singh/VSD_IAT_Sign-off_Timing_Analysis-Basics_to_Advanced/blob/main/Theory.md#day-4-summary)

# Day-4 Labs
  ## Understanding clock gating check
  ![Screenshot from 2022-02-07 23-37-03](https://user-images.githubusercontent.com/73732594/152846371-b79e12aa-b53e-466b-acff-fea2b30c7797.png)

  ![Screenshot from 2022-02-07 23-29-38](https://user-images.githubusercontent.com/73732594/152846178-92c2bc06-8966-437f-a476-d6a535481247.png)

  ## Understanding Async pin checks
  
  ![Screenshot from 2022-02-07 23-40-08](https://user-images.githubusercontent.com/73732594/152846814-6e7fc775-c548-4aca-b30e-264700bd9f6a.png)

  ![Screenshot from 2022-02-07 23-41-26](https://user-images.githubusercontent.com/73732594/152847251-387876f0-df1d-4bd1-8fe0-735e81107fa4.png)
  
  ![Screenshot from 2022-02-07 23-42-41](https://user-images.githubusercontent.com/73732594/152847313-5bad3d31-481b-430d-b235-5210887bcb26.png) 

OpenTimer 2.1.0 <br>
Time unit        : 1e-12 s <br>
Capacitance unit : 1e-15 F <br>
Voltage unit     : 1 V <br>
Current unit     : 0.001 A <br>
Power unit       : 1e-06 W <br>
Pins           : 22 <br>
POs            : 1 <br>
PIs            : 6 <br>
Gates          : 3 <br>
Nets           : 19 <br>
Arcs           : 25 <br>
SCCs           : 0 <br>
Tests          : 6 <br>
Cells          : 105 <br>

# Day-5 Summary

[Day-5 Summary](https://github.com/Sumanyu-Singh/VSD_IAT_Sign-off_Timing_Analysis-Basics_to_Advanced/blob/main/Theory.md#day-5-summary)

# Day-5 Labs
  ## Revisit slack computation
  ![Screenshot from 2022-02-07 23-48-15](https://user-images.githubusercontent.com/73732594/152848018-392cb1af-684d-4ef0-9329-735edb5ddcb1.png)

  ## Clock Reconvergence Pessimism (CRP) Basics
  ![Screenshot from 2022-02-07 23-50-44](https://user-images.githubusercontent.com/73732594/152848433-4f1a25bf-72fd-412b-8bf7-99f640e3a2ce.png)

  ## Clock Reconvergence Pessimism Removal (CRPR)
  ![Screenshot from 2022-02-07 23-52-54](https://user-images.githubusercontent.com/73732594/152848638-09978f8d-a4ad-40e7-805c-a0c47cc9c6a0.png)
  
  ## Engineering Change Order (ECO)

## Acknowledgements

- [Kunal Ghosh](https://github.com/kunalg123), Co-founder of VLSI System Design (VSD) Corp. Pvt. Ltd.
- [Vikas Sachdeva](https://vlsideepdive.com/), Advisor, Tech and VLSI Coach, Trainer and Innovator at vlsideepdive.

## Author

[Sumanyu Singh](https://www.linkedin.com/in/sumanyu-singh-718272170/), M.Tech (Microelectronics), IIITA (2021-23), IIIT Allahabad, Prayagraj, Uttar Pradesh, India












 

 
 










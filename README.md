# VSD-IAT: Sign-off Timing Analysis - Basics to Advanced
![vsd_iat_logo](https://user-images.githubusercontent.com/73732594/152016610-be3ef4c8-601c-40e7-af85-91dc3ae9b2a4.png)

Static Timing Analysis (STA) is a method of verifying the timing performance of a digital circuit design by analyzing the delays of the circuit's logic paths. It is typically used to ensure that the circuit will meet its timing requirements, such as setup time, hold time, and clock-to-output delays. 

It starts with basics of Static Timing Analysis, timing paths, startpoint, endpoint and combinational logic definitions. It explains setup and hold checks, how STA tools calculate setup and hold violations. Then it slowly builds up to cover all aspects of STA like multiple types of timing paths, design rule checks, checks on async pins and clock gates. After that we go into slightly advanced topics like Time borrowing on latches, timing arcs, cell delays and models, impact of clock network on STA. Since STA and timing constraints go hand in hand the workshop covers basics of all the timing constraints that an engineer should know for STA like clock definitions, clock groups, clock characteristics, port delays and timing exceptions. 

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
  * [Understanding full reg to reg STA analysis](#understanding-full-reg-to-reg-sta-analysis)
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

Static Timing Analysis (STA) is a method of verifying timing performance of a design. Its key feature is that it is exhaustive compared to functional simulation and SPICE simulation. It doesn't require testbenches and is not used for asynchronous design. It doesn't rely on imput vectors and is a mathematical technique going through all the paths. It doesn't verify functionality of the design. Its pessimistic and conservative so as to ensure there is a definite guard band for sign-off. Typical inputs for STA are netlist, SDC or constraints file, and logic libraries.

STA breaks the paths at ports and sequential elements.

![timing_paths](https://user-images.githubusercontent.com/100671647/220118112-387f4ce6-eb1f-4948-ac85-d19279178393.png)

**Different timing paths are shown above**

The timing analysis is performed on each of these paths.

Broadly speaking, there are three components of the path, which are, startpoint, endpoint and combinational logic.

Startpoint is where the data is launched by clock edge, or where the data must be available at a specific time. It is usually input port or register clock pinhe clock pin is taken into account because CLK -> Q delay also comes into picture.

Endpoints are where data is captured by clock edge, or where the data must be available at a specific time. It is usually output port or register data pin.

Combinational logic blocks are elements which have no memory, or internal state. Ex.: AND gate, OR gate.

Between a set of startpoint and endpoint, combinational logic might have multiple paths.

![set_up_chk](https://user-images.githubusercontent.com/100671647/220123544-1e8c8906-035b-4cfa-9f12-88f61ed96d02.png)

**Data and clock signals for setup check shown above (In first, setup time is met and in 2nd it gets violated)**

![hold_check](https://user-images.githubusercontent.com/100671647/220124281-879b921b-2532-4235-bde0-d608ef95726e.png)

**Data and clock signals for hold check shown above (In first, hold time is met and in 2nd it gets violated)**


Setup time of a flop is dependent on the technology node. Value is available in logic libraries. This enforces max delay on the data path. Data should be available at the input of sequential device before clock edge that captures the data.

Similarly hold time enforces min delay on the data path. Here, data should be stable at the input of sequential device for sometime after the clock edge that captures the data.

Setup Slack = Data Required Time - Data Arrival Time

Slack is positive when data arrives earlier than required time.
![pos_slack](https://user-images.githubusercontent.com/100671647/220125981-e5839030-1dcd-47cf-9cba-c4439581a03e.png)

**Above picture shows positive slack**

Slack is negative when data arrives later than required time.

Timing is met when slack is positive, otherwise it isn't.

Synopsys Design Constraints (SDC) for timing specify parameters affecting operational frequency of the design. Examples: create_clock, create generated_clock, set_clock groups, set_clock_transition, set_timing_derate, etc.
![create_clk](https://user-images.githubusercontent.com/100671647/220128930-b49dab46-716c-439d-917a-5841fe147a5f.png)

**Some examples of create_clock and create_generated_clock command are shown above**


Similarly, there are constraints for area and power, which specify restrictions about the area and power. Examples: set_max_area, set_max_dynamic_power, etc. and constraints for design rules which are requirements of the target technology. Examples: set_max_capacitance, set_max_transition, set_max_fanout, etc. 

There are exceptions to design constraints which relax the requirements set by other commands. Examples: set_false_path, set_multicycle_path, etc.

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

set_input_delay 5 –max –rise [get_ports inp1] –clock tau2015_clk 

set_output_delay -10 –min –fall [get_ports out] –clock tau2015_clk

## OpenSTA Run script

Below are the commands used for running the STA flow:

![run_script](https://user-images.githubusercontent.com/100671647/220426436-46f173a7-6977-4f44-a773-5996c84e4e2d.png)

For running the OpenSTA, we use below command

	sta run.tcl -exit | tee run1.log

Below are the reports after running OpenSTA:

![violated](https://user-images.githubusercontent.com/100671647/220427581-bbc782a8-e9e3-4269-9ed7-598a69a50212.png)
![met](https://user-images.githubusercontent.com/100671647/220427821-abb25d94-5f2b-44c9-8176-be51baffe25c.png)

In 1st, the timing checks aren't met(VIOLATED) since the slack is negative but in 2nd it is MET (slack is positive).

# Day-2 Summary

Apart from setup and hold checks, STA also has other timing checks in place like clock gating checks, async pin checks and data to data checks. We also make a note of the rise and fall slew transitions. We also have to provide load analysis by specifying min and max capacitances on the IO net, and corresponding fanout load on ports and output pins. The skew between launch and capture clock waveforms also needs to be taken into account, the skew is positive if the capture flop clock leads the launch flop clock. The duty cycle of the clock is limited by various parameters apart from the technology node. Latch based designs allow more flexibility in timing and also aids time borrowing. A typical STA text report contains startpoint, endpoint, 'max' signifies setup time check and the nodes in the design mentioned as paths, whose respective delays are taken into account.   

# Day-2 Labs

## Liberty File and Understanding .lib file
A "liberty file" is a file format that provides timing and power information for cells in a standard cell library. A standard cell library is a collection of pre-designed cells, such as gates and flip-flops, that can be used to create digital circuits.

The liberty file contains timing information about the delay of each cell and its input and output pins. This information is critical for accurate timing analysis of a circuit. It also provides information on the power consumption of each cell, which is important for power analysis and optimization.

The .lib file is an ASCII representation of the timing and power
parameters associated with any cell in a particular semiconductor
technology The .lib file contains timing models and data to calculate I/O delay paths, Timing check values and Interconnect delays.

**Exercises:**

• Find all the cells in simple_max.lib.
• Find all the pins of the cell NAND2_X1 in simple_max.lib

211 and 3.

• What difference you see between NAND2_X1 and NAND3_X1

The max capacitance increases multifold, and the number of input pins.

• What is the difference between ‘simple_max.lib’ and ‘simple_min.lib

Fabrication process variations could either increase or decrease the delay of a cell. So we need to set early and late value while setting the derate factor. STA tool would consider early or late timing derate based on the path and type of analysis.

## Understanding Lib Parsing



![Screenshot from 2022-02-07 21-21-25](https://user-images.githubusercontent.com/73732594/152823092-c6aac9a5-1b1c-4bc6-911a-728a2949ffa0.png)


## Understanding SPEF file and SPEF parsing

In static timing analysis (STA), a "spef" file (also known as a Standard Parasitic Exchange Format file) is a commonly used file format that describes the parasitic capacitance and resistance of a digital circuit.

During STA, a spef file is used to model the interconnect delays and timing constraints of a design. The file contains information about the routing topology of the design, including the location and size of interconnect wires, the capacitance and resistance of the wires, and the location and size of the devices (gates and pins) in the design.

By analyzing the spef file, STA tools can estimate the timing delays of the interconnect wires and the devices, and determine if the design meets its timing requirements. The spef file is typically generated by layout tools during the physical design stage of a chip.

SPEF can be generated by place-and-route tool or a parasitic extraction tool, and then this SPEF is used by timing analysis tool for checking the timing, in-circuit simulation or to perform crosstalk analysis.

Parasitics can be represented at many different levels.
A Typical SPEF File has 4 main sections

1. Header
2. Name Map
3. Top Level Ports
4. Parasitic description



![Screenshot from 2022-02-07 22-37-20](https://user-images.githubusercontent.com/73732594/152836739-3295abdf-d8a3-4b8b-812b-cd9b4fd2a36a.png)


## Understanding Timing Reports 

![Screenshot from 2022-02-07 22-37-20](https://user-images.githubusercontent.com/73732594/152837812-50e222c9-106d-472f-94c2-2e2d5b9341f3.png)
This is the timing report we have obtained from the netlist and other inputs being parsed into the OpenTimer tool.

![Screenshot from 2022-02-07 22-53-31](https://user-images.githubusercontent.com/73732594/152839697-85913598-8ed4-49df-8d95-8e3cb26ad87c.png)

# Day-3 Summary

When we have multiple clocks, in STA, a possible common base period is choses, and a restrictive setup and hold check is followed. By default, the checks are restrictive in nature. The inclusion of falling edge clock on capture after positive edge on launching flop may lead to a half cycle period delay. Timing arcs consist of cell and net arcs. Combinational arcs go from input pins to the output pin, whereas sequential arcs consist of CLK->D arc (Setup/hold arc), CLK->OUT (Propagation delay) and CLK->RST (Recovery/Removal). We are also introduced to the concept of unateness and non-unateness. Cell delays are in general fuction of input transition and capacitive load. Clock latency can be from the source and towards the network. Unlike ideal clocks, real clocks have jitter which leads to uncertainty of the position of rising/falling edge.

# Day-3 Labs

## Understanding full reg to reg STA analysis

![Screenshot from 2022-02-07 23-16-53](https://user-images.githubusercontent.com/73732594/152843403-144f3783-6c80-4160-87dc-e907c2843bb3.png)
![Screenshot from 2022-02-07 23-18-51](https://user-images.githubusercontent.com/73732594/152843754-f891934b-ab2d-4390-bba9-f6f8fc21783b.png)

This is the netlist and sdc file utilized for the analysis.

![Screenshot from 2022-02-07 23-03-03](https://user-images.githubusercontent.com/73732594/152841252-d13330d3-4576-46bd-a201-4d73e738630d.png)

Functionally, it might seem like the first path, corresponds to the slack computation but the same cannot be mapped from the report exactly.


## Understanding Slack computation

![Screenshot from 2022-02-07 23-07-07](https://user-images.githubusercontent.com/73732594/152842219-27d77509-e33c-4101-80a9-44b4c3acb55a.png)

![Screenshot from 2022-02-07 23-09-25](https://user-images.githubusercontent.com/73732594/152842243-401bf7ff-c1e7-4bc6-97b1-e2c8348d17b4.png)

# Day-4 Summary

Crosstalk may lead to delta delays and glitches. Switching activity affected by coupling of aggresor and victim, leads to delta delay which may cause timing failure. Glitching is caused due to sudden charge transfer on a stable net which may cause functional failure. Variation could be inter-die and intra-die. The former is of global and systematic, and latter is of on-chip and random nature. In clock gating transition on gating pin, shouldn't create unnecessary active edge of the clock in the fanout. Async pin checks are needed to avoid unknown state. Recovery and removal time checks for assertion and deassertions need to be checked therefore.


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

In Synchronous clocks, events happen at a fixed phase relation whereas in asynchronous clocks that is untrue. Then there are logically exclusive clocks, which are passed through a mux logic, whereas in physically exclusive clocks, the sources are entirely different. set_clock_groups is used for establishing asynchronous and synchronous pairs. We can provide different properties like latency, uncertainty, transition and sense. Path specification is done by providing -from -to and -through flags. We would need to be careful of false paths and multicycle paths too. There is a provision of providing max and min delay for a path too.

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












 

 
 










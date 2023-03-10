* [Day-1 Summary](#day-1-summary)

* [Day-2 Summary](#day-2-summary)

* [Day-3 Summary](#day-3-summary)

* [Day-4 Summary](#day-4-summary)

* [Day-5 Summary](#day-5-summary)


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

# Day-2 Summary

Apart from setup and hold checks, STA also has other timing checks in place like clock gating checks, async pin checks and data to data checks. We also make a note of the rise and fall slew transitions. We also have to provide load analysis by specifying min and max capacitances on the IO net, and corresponding fanout load on ports and output pins. The skew between launch and capture clock waveforms also needs to be taken into account, the skew is positive if the capture flop clock leads the launch flop clock. The duty cycle of the clock is limited by various parameters apart from the technology node. Latch based designs allow more flexibility in timing and also aids time borrowing. A typical STA text report contains startpoint, endpoint, 'max' signifies setup time check and the nodes in the design mentioned as paths, whose respective delays are taken into account.   

**Other Timing Checks**
Apart from Hold and Setup checks(which happens in data pins with respect to data pins) STA also dose other types of check like 
- Clock Gating Checks(done on clock enable pin with respect to the clock pin)
- Async Pin Checks(checks when reset pin, set pin, clear pin can be inserted with respect to clock) 
- Data to Data Checks(identify may skew between 2 pins)
    
![other_timing](https://user-images.githubusercontent.com/100671647/220512531-b57a8054-4125-4dd7-9970-5d01c9a2d661.png)

**Design Rule Checks**

Design Rule Checks specifies about
1. Slew/Transition Analysis
  
2. Load Analysis
- We can specify minimum and maximum capacitance on ports and nets.
- Also specify the maximum fanout load on ports and output pins.
      
3. Clock Skew Analysis
- It is difference in delay of the clock at different points.
- It is basically the delay between the launch clock and capture clock.
- Skew is said to be positive when caputre clock has more delay.
    
4. Pulse Width Checks
- As clock travels through the path the clock pulse can change(can happen when rise delay and fall delay are not same) called as Shrinked Clock.
- If this shrinked clock is below the certain value then pulse width violation happens.
    
![drcs_n_pwc](https://user-images.githubusercontent.com/100671647/220513015-ca6925ec-92e8-4363-8a65-c7a4098aec69.png)

**Latch Timing:**
**Latch can start sampling data from the rising edge(or falling edge) itself and continue sampling till the respective falling edge (or rising edge)** because latches are level sensitive whereas **Flipflop can only sample the data "at" Rising edge or Negative edge** because flipflops are edge sensitive. Both holds the data when they are disable (Latch disable at level and Flipflop disable just after triggering level).

- In case of STA, Enable pin of the latch acts like a clock to a latch(as latches do not have clock input).
    
Generally designers prefer flip flops over latches because of this edge-triggered property, which makes their life easy to do analysis and interpretation the design. 
    
Latch-based designs are preferred in case of clock frequency in GHz (in high-speed designs). In flip-flop-based high-speed designs, maintaining clock skew is a problem, but latches ease this problem.
    
**Time Borrowing** is the property of a latch by virtue of which a path ending at a latch can borrow time from the next path in pipeline such that the overall time of the two paths remains the same. The time borrowed by the latch from next stage in pipeline is, then, subtracted from the next path's time.

![time_borrow](https://user-images.githubusercontent.com/100671647/220513304-3218e6a7-b031-4cab-a3be-bb75f136551b.png)


# Day-3 Summary

When we have multiple clocks, in STA, a possible common base period is choses, and a restrictive setup and hold check is followed. By default, the checks are restrictive in nature. The inclusion of falling edge clock on capture after positive edge on launching flop may lead to a half cycle period delay. Timing arcs consist of cell and net arcs. Combinational arcs go from input pins to the output pin, whereas sequential arcs consist of CLK->D arc (Setup/hold arc), CLK->OUT (Propagation delay) and CLK->RST (Recovery/Removal). We are also introduced to the concept of unateness and non-unateness. Cell delays are in general fuction of input transition and capacitive load. Clock latency can be from the source and towards the network. Unlike ideal clocks, real clocks have jitter which leads to uncertainty of the position of rising/falling edge.

**Multiple Clocks:** When different types of clock are present(of different frequency) then setup check is calculated or the most restrictive setup is identified by first expanding a clock by a **common base period** and in that whichever is the closest launch and capture which is used find the most restrictive setup. The hold check is dependent upon the setup check.

**Setup and Hold Check**

![setup_hold_chk](https://user-images.githubusercontent.com/100671647/220534395-fae1ed3d-c9b7-4f3e-8c5e-47e2dabc1566.png)

Two rules for **Hold Check**

**1.**   

![rule_1](https://user-images.githubusercontent.com/100671647/220515371-45e7ad16-81da-44c8-b37d-e00644d29214.png)

**2.**

![rule_2](https://user-images.githubusercontent.com/100671647/220515779-c6f16ea8-c20f-4622-b460-ce879a9fe99d.png)

   
**Two types of Timing Arch** 

1. **Cell Arch** - It is the input to output connection in the cell present in logical libraries . These arch will have information about the input-output functionality, rise and fall time, delays, etc. 

2. **Net Arch** - It connects one cell to other cell and there are ways to calculate the information about the cell to cell rise and fall time, delays, etc.

![archs](https://user-images.githubusercontent.com/100671647/220516180-39753771-5610-4b2b-abfd-72875bb1e656.png)

**Combinational and Sequential Archs**

![com_n_seq_arch](https://user-images.githubusercontent.com/100671647/220516520-e2ae3947-9a78-429a-813f-9c563e955ce5.png)

**Positive, Negative and Non Unate Arch**

**Positive Unate Arch** follows the input in same direction. It basically tells us that when input rises the output also rises or when input falls the output also going to fall. **Negative Unate Arch** follows the input in opposite direction. It basically tells us that when input rises the output falls or when input rises the output also going to fall. In **Non Unate Arch** output cant be predicted when input changes.
    
![pos_neg_non_unate_arch](https://user-images.githubusercontent.com/100671647/220517519-88fdd12c-a4e3-4b2d-a403-6c7803ad0168.png)

**Cell Delays:** Cell Arch have these Cell Delays 
    
![cell_delays](https://user-images.githubusercontent.com/100671647/220517750-0a1bc8dc-6869-4b55-b82f-4b46531abefa.png)

**Clock Skew** is the time difference between arrival of the same edge of a clock signal at the Clock pin of the capture flop and launch flop. It is basically the delay difference between the two clocks.
    
**Positive and Negative Skew**

![pos_neg_skew](https://user-images.githubusercontent.com/100671647/220528422-7b53e8c3-e424-431a-bab8-fb928d179830.png)

- So because of the positive skew we got the extra time to meet setup and it makes setup time easer to meet, whereas to meet hold time we need to keep the data stable for more time. 
 
- So because of the negative skew we got the less time to meet setup and it makes setup time difficult to meet, whereas to meet hold time we need to keep the data stable for less time. 
 
**Clock Latency:** The time taken by Clock signal to reach from clock source to the clock pin of a particular flip flop is called as Clock latency.

![clk_latency](https://user-images.githubusercontent.com/100671647/220528882-24a3d88b-56c0-439b-8751-9853db790ac5.png)


**Clock Jitter:** is a characteristic of the clock source and the clock signal environment. It can be defined as ???deviation of a clock edge from its ideal location.??? Clock jitter is typically caused by clock generator circuitry, noise, power supply variations, interference from nearby circuitry etc.

![clk_jitter](https://user-images.githubusercontent.com/100671647/220533943-f1266a50-122b-4a41-82fe-af86c0f137b7.png)

**Setup Check and Setup Check With Clock Skew and Jitter**

![set_up_chk_with_jitter](https://user-images.githubusercontent.com/100671647/220529986-686a765d-8a45-4db0-8f1f-7a4b1f07b71a.png)

![image](https://user-images.githubusercontent.com/100671647/220530944-1351f0fb-0a3c-4e7c-be0f-05ef6c17fee2.png)

 
**Hold Check and Hold Check With Clock Skew and Jitter**

![hold_chk_with_jitte](https://user-images.githubusercontent.com/100671647/220530542-ae96f236-a05b-4e75-bee4-e1108d0ad996.png)

![image](https://user-images.githubusercontent.com/100671647/220531097-fc1fa03a-7d9d-4b3e-ac4d-d404a40ec3a9.png)


**Clock Reconvergence Pessimism (CRP):** It is a term used in STA to describe a scenario where the actual timing of a circuit is better than what is predicted by the STA tool. This happens because STA assumes that all signals will be delayed by the same amount of time from their arrival at the inputs to their arrival at the outputs, regardless of the timing relationship between them.

- Clock reconvergence pessimism (CRP) is a delay difference between the launching and capturing clock pathways. The most prevalent causes of CRP are convergent pathways in the clock network and varying minimum and maximum delays of clock network cells.  
 

- Clock reconvergence pessimism specifically refers to the situation where there are multiple clock signals in the circuit, and the STA tool assumes that they will all arrive at the same time, even though in reality they may have different arrival times due to variations in the physical layout of the circuit. This can lead to an overestimation of the required delay between different parts of the circuit, resulting in a slower circuit than necessary.

**CRPR-Clock Reconvergence Pessimism Removal:** To account for clock reconvergence pessimism, designers may add extra delay to the clock tree or use a more advanced STA tool that can model clock skew more accurately. It is important to reduce clock skew to minimize the impact of clock reconvergence pessimism on the circuit's performance.
 
![crpr_removal](https://user-images.githubusercontent.com/100671647/220535101-edb62731-1dcf-4967-962b-20686d0d9a77.png)

**STA Text Report:** When STA tool dose analysis and reports setup and hold timing, it first converts that logic into nodes and cells and archs.
  
![logic_to_cell_arch](https://user-images.githubusercontent.com/100671647/220539019-baffee1a-0dbc-4dda-b56d-f5eb93b6a5f2.png)

**Sample STA Text Report**

![txt_report_sample](https://user-images.githubusercontent.com/100671647/220539398-bc4ffbf1-a7d5-4d79-8dcf-7515011dc84c.png)

# Day-4 Summary

Crosstalk may lead to delta delays and glitches. Switching activity affected by coupling of aggresor and victim, leads to delta delay which may cause timing failure. Glitching is caused due to sudden charge transfer on a stable net which may cause functional failure. Variation could be inter-die and intra-die. The former is of global and systematic, and latter is of on-chip and random nature. In clock gating transition on gating pin, shouldn't create unnecessary active edge of the clock in the fanout. Async pin checks are needed to avoid unknown state. Recovery and removal time checks for assertion and deassertions need to be checked therefore.

**Crosstalk and Noise**

In STA, crosstalk and noise can affect the timing of signals and ultimately impact the performance and reliability of the digital circuit.

Crosstalk refers to the phenomenon of a signal on one wire interfering with the signal on an adjacent wire. This can happen due to capacitive coupling between the wires, and it can lead to errors in signal timing. Crosstalk can cause a signal to arrive at a destination later or earlier than expected, which can result in setup and hold time violations. In static timing analysis, crosstalk is typically modeled by adding delays to the timing paths in the affected nets.

Noise refers to any unwanted variation in the signal that is not related to the desired signal. In digital circuits, noise can be caused by a variety of sources, such as power supply fluctuations, ground bounce, and electromagnetic interference. Noise can also lead to errors in signal timing, similar to crosstalk. In static timing analysis, noise is typically modeled by adding jitter to the arrival times of signals.

**Crosstalk and Noise Removal:** To account for crosstalk and noise in static timing analysis, designers typically use conservative timing constraints and design margins. Additionally, they may use specialized tools to model and analyze crosstalk and noise effects, such as signal integrity analysis tools.
    
![cross_talk](https://user-images.githubusercontent.com/100671647/220541316-cbd7263e-c12c-4b72-a9c3-59c3c6bd9772.png)

- In this when both aggressor is changing from 0 to 1 and victim is also changing in the same direction so it can couse the victim signal to change faster, hence rise time changes and **delay decreases**
- If the aggressor and victim are changing in opposite direction then because of coupling caps aggressor will try to push the victim signal in same direction, hence chage becomes slower in victim which in **increases delay**
- STA take this changes into consideration and accordingly calculate setup and hold time.
    
![cross_talk_glitches](https://user-images.githubusercontent.com/100671647/220541761-40bd520b-d95f-4232-ad43-c672a260b60c.png)

- When aggressor is changing whereas victim signal in not changing so we can have glich in the victim signal which could lead to functional failure. So STA need to make sure these glitches are not present or should be below a threshold value.
    
**Process Variation**

- Sometimes we have process variation(can have differnt delay or transition time)within a chip or from wafer to wafer or within the wafer. So STA need to take this into account.
    
![pv](https://user-images.githubusercontent.com/100671647/220542088-a3acfc19-c77b-451b-aeef-085184d85e3a.png)
    
### Clock Gating Checks 

Clock gating is a technique used in digital circuit design to reduce power consumption by selectively turning off clock signals to unused circuit blocks. However, clock gating can also have an impact on the timing of a design.

There are two main types of clock gating checks in STA:

- 1. Clock gating analysis: This type of check analyzes the clock gating logic to ensure that it meets the design's timing requirements. The analysis considers the setup and hold times of the gated signals as well as the delay through the clock gating logic itself.

- 2. Clock gating optimization: This type of check looks for opportunities to optimize the clock gating logic to improve the overall timing of the design. The optimization may involve changing the gating logic or adjusting the timing constraints to make better use of the clock gating.


- A clock gating check occurs when a gating signal can control the path of a clock signal at a logic cell. For example when a clock and enable signal is fed as input to an AND gate.
- The signal must be used as clock downstream, like feed a flop or latch clock pin or feed output port or feed generated clock.
- Intention of this check is that transition on gating pin does not Create unnecessary active edge of the clock in the fanout.

**Active High and Active Low Clock Gating Check**

![image](https://user-images.githubusercontent.com/100671647/220543032-0340673c-b53b-433c-899d-8acabb0b50f8.png)

- So, In case of AND and NAND gates, clock gating checks that during setup check the enable should become stable sometime before the clock rising edge(so that coack has enough setup time) and during hold check the enable should become stable sometime after the clock rising edge.
   
- And, In case of OR or NOR gates, clock gating checks that before some time and after some time of the low value of the clock enable pin should be stable.
    
Sometimes tools cannot set this clock gating checks and gives error, so we need to do it manually by using this command
```
   set_clock_gating_check
```
  
**Checks on Async Pins**

- Async Pins in the designes are reset or clear pins 
- These checks are needed only of asynchronous pins not for synchronous pins(beacure synchronous reset pins come from flops which is alraedy synchronised )
    
![async_pin_checks](https://user-images.githubusercontent.com/100671647/220543540-a2119665-1216-4106-bec6-1cc5365db624.png)
- Assertion means clear is on and outputs are zero, which is indepeant of clock
- De-assertion means clear is off and output is dependent on input and clock.
    
![asser_deasser](https://user-images.githubusercontent.com/100671647/220544149-3eeeb5ca-4087-48c1-8fca-2824ac374e7b.png)

- Assertion to De-assertion change should not happen very close to clock edge, it should happen sometime before clock edge and sometime after clock edge which is known as the **Recovery Time** and this time requirement is given by the logival liblary of the flop.

# Day-5 Summary

In Synchronous clocks, events happen at a fixed phase relation whereas in asynchronous clocks that is untrue. Then there are logically exclusive clocks, which are passed through a mux logic, whereas in physically exclusive clocks, the sources are entirely different. set_clock_groups is used for establishing asynchronous and synchronous pairs. We can provide different properties like latency, uncertainty, transition and sense. Path specification is done by providing -from -to and -through flags. We would need to be careful of false paths and multicycle paths too. There is a provision of providing max and min delay for a path too.

**Different types of Clock Groups**

**Synchronous and Asynchronous Clocks**

![syn_asyn_clocks](https://user-images.githubusercontent.com/100671647/220545474-2cc6247a-60f4-4e15-acd0-ba3f33a47980.png)

**Logically and Physically Exclusive Clocks**

![logi_phy_excl_clocks](https://user-images.githubusercontent.com/100671647/220546012-c5e59176-d839-4d36-b6da-ef516d1c8c7b.png)

"**set_clock_groups**" command: By default STA assumes synchronous clocks.
![set_clk_grp](https://user-images.githubusercontent.com/100671647/220546439-174097b8-1bc2-4033-924a-2966c4d46da2.png)

- Below picture shows what happens When only one group is specified:

![only_one_grp](https://user-images.githubusercontent.com/100671647/220546818-0ab95b66-a833-4c9a-8889-aca7228dd085.png)

Multiple **Clock Properties** are shown below:

![clk_prop](https://user-images.githubusercontent.com/100671647/220547168-52b07596-d339-4ac8-9c62-e3cd308e0e92.png)

**Timing Exceptions:** These are particular set of commands in STA that can modify the default behaviour.
Before that, let's understand "Path Specification". As there could be multiple paths between start point and end point and we want to change the default behaviour of a particular path. So, we need to specify that path.

![path_specif](https://user-images.githubusercontent.com/100671647/220549282-b10777ca-46f4-4a13-af49-37d68aa6b648.png)

Path Specification is done by typically three commands:
- from (start points: either ports or typically clock pin of flops in design)
- to (end point)
- through (nodes between start point and end point)

"**set_false_path**" command tells STA tool that don't time this path.
Example: from(F1/CK)-through(U1/Z)-to(F5/D)

![set_false_path](https://user-images.githubusercontent.com/100671647/220551170-d691ee9d-35a4-49cf-b4e3-dc59c9875c73.png)

"**set_multicycle_path**" command is used to change the default behaviour.Command shown in below picture tells STA tool to as setup check in two cycle.

![mul_cycle_path](https://user-images.githubusercontent.com/100671647/220596357-f2533576-3e9e-4723-a11e-40b61dadfc84.png)

**Other Exceptions are:**

![image](https://user-images.githubusercontent.com/100671647/220596824-4038c91b-5561-42a5-974f-c6f9c10df4ba.png)

**Multiple Modes:**
"**set_case_analysis**" command is used to specify a certain portion of the design and specify a constant value. That net may or may not be a constant value in the design netlist but we can overwrite it using "set_case_analysis" command.

For Example: We have a 2:1 mux in our design with sel(select line), Tclk (In0), Fclk(In1) as Inputs and we want that Fclk(functional clock) should be propagated. We can do this by "set_case_analysis" command by setting sel=1. Note that we have not connected sel=1 in the netlist.


**ECO ??? Engineering Change Order**

??? In the ECO cycle, we perform various analysis one by one for every 
check which we need to close but not closed till PnR stage. 

??? There are specialized signoff tools that help us to analyze the issue 
and also suggest the changes we need to do in order to close the 
issue. 

??? The suggested change is captured in an eco file.

??? In this lab we will focus on ECO for timing purposes, this is done to fix 
setup and hold violation.




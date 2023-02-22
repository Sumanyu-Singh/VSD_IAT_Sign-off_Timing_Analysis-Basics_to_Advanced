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

![setup_chk](https://user-images.githubusercontent.com/100671647/220515106-0c4a68ac-9842-43b8-a31a-58e8994af3e9.png)

![hold_chk](https://user-images.githubusercontent.com/100671647/220515581-5490a958-683e-4d97-a949-41d4f5ee70cd.png)

Two rules for **Hold Check**

1.   

![rule_1](https://user-images.githubusercontent.com/100671647/220515371-45e7ad16-81da-44c8-b37d-e00644d29214.png)

2.

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


**Clock Jitter:** is a characteristic of the clock source and the clock signal environment. It can be defined as “deviation of a clock edge from its ideal location.” Clock jitter is typically caused by clock generator circuitry, noise, power supply variations, interference from nearby circuitry etc.

![clk_jitter](https://user-images.githubusercontent.com/100671647/220529452-d8bab160-1675-46b0-a89d-a232d39fe761.png)

**Setup Check and Setup Check With Clock Skew and Jitter**

![set_up_chk_with_jitter](https://user-images.githubusercontent.com/100671647/220529986-686a765d-8a45-4db0-8f1f-7a4b1f07b71a.png)

![image](https://user-images.githubusercontent.com/100671647/220530944-1351f0fb-0a3c-4e7c-be0f-05ef6c17fee2.png)

 
**Hold Check and Hold Check With Clock Skew and Jitter**

![hold_chk_with_jitte](https://user-images.githubusercontent.com/100671647/220530542-ae96f236-a05b-4e75-bee4-e1108d0ad996.png)

![image](https://user-images.githubusercontent.com/100671647/220531097-fc1fa03a-7d9d-4b3e-ac4d-d404a40ec3a9.png)


    
### CRPR-Clock Reconvergence Pessimism Removal
 
![Screenshot (2479)](https://user-images.githubusercontent.com/120498080/220192392-95da6514-59d8-4362-b9d3-806b0c0a5d5a.png)
- During decision of max and min launch path sometimes there is confussion in choosing the best path because some part of the circuit is common in two paths and that can be soled by CRPF    
- Clock reconvergence pessimism (CRP) is a delay difference between the launching and capturing clock pathways. The most prevalent causes of CRP are convergent pathways in the clock network and varying minimum and maximum delays of clock network cells.  
 

### STA Text Report
When STA tool dose analysis and reports setup and hold timing, its first converts that logic into nodes and cells and archs.
  
![Screenshot (2438)](https://user-images.githubusercontent.com/120498080/220064935-353bce95-62b9-43e4-81ae-871a6a9e0994.png)

#### Sample STA Text Report
![Screenshot (2441)](https://user-images.githubusercontent.com/120498080/220065230-b7350659-e6a7-4edc-92ac-ae32057a372f.png)  

# Day-4 Summary

Crosstalk may lead to delta delays and glitches. Switching activity affected by coupling of aggresor and victim, leads to delta delay which may cause timing failure. Glitching is caused due to sudden charge transfer on a stable net which may cause functional failure. Variation could be inter-die and intra-die. The former is of global and systematic, and latter is of on-chip and random nature. In clock gating transition on gating pin, shouldn't create unnecessary active edge of the clock in the fanout. Async pin checks are needed to avoid unknown state. Recovery and removal time checks for assertion and deassertions need to be checked therefore.

# Day-5 Summary

In Synchronous clocks, events happen at a fixed phase relation whereas in asynchronous clocks that is untrue. Then there are logically exclusive clocks, which are passed through a mux logic, whereas in physically exclusive clocks, the sources are entirely different. set_clock_groups is used for establishing asynchronous and synchronous pairs. We can provide different properties like latency, uncertainty, transition and sense. Path specification is done by providing -from -to and -through flags. We would need to be careful of false paths and multicycle paths too. There is a provision of providing max and min delay for a path too.


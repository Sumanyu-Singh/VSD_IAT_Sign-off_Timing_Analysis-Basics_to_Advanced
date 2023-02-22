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

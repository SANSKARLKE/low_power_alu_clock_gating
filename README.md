# Post-Implementation Power Optimization of a Clock-Gated ALU on FPGA Fabric

This project implements and validates a **32-bit hierarchical Arithmetic Logic Unit (ALU)** in VHDL optimized through **architectural clock gating** on an AMD Xilinx Artix-7 FPGA (`xc7a100tcsg324-1`). Unlike a standard un-gated ALU where clock edges continuously drive all internal registers regardless of the active instruction, this design partitions the clock network using dedicated `BUFGCE` primitives controlled by immediate opcode decoding, disabling clock transitions to dormant functional blocks to eliminate redundant switching dissipation.

Verified at an operating frequency of **50 MHz** using post-implementation Switching Activity Interchange Format (SAIF) back-annotation across a **6,460 ns** high-stress stimulus workload, the clock-gated ALU achieves a **17.65% reduction in dynamic power** (dropping from 17 mW to 14 mW, and lowering total on-chip power from 108 mW to 105 mW) compared to the baseline architecture[cite: 11]. This improvement is driven by a **66.67% reduction in internal logic power** (from 3 mW to 1 mW) and a **40.00% decrease in internal signal dissipation** (from 5 mW to 3 mW)[cite: 11], while maintaining robust timing closure with a positive Worst Negative Slack (WNS) of **+6.537 ns**.

## Steps to Verify

1. **Synthesize and Implement:** Open the `.xpr` project file in Vivado, run **Synthesis**, and complete **Implementation** to generate the placed-and-routed netlist.
2. **Run Post-Implementation Simulation & Generate SAIF:** Launch **Run Simulation > Run Post-Implementation Functional Simulation**, then execute the following in the **Tcl Console** to log the 6,460 ns high-stress testbench (`tb_top_alu.vhd`) from time zero:
   ```tcl
   restart
   set proj_dir [get_property DIRECTORY [current_project]]
   open_saif "$proj_dir/switching_activity.saif"
   log_saif [get_objects -r /tb_top_alu/UUT/*]
   run 6460 ns
   close_saif
   ```
3. **Verify Dynamic Power:** Open the **Implemented Design** and run the commands below in the **Tcl Console** to back-annotate the SAIF file and confirm the dynamic power drop from 17 mW (standard) to 14 mW (clock-gated) in the **Power** report[cite: 11]:
   ```tcl
   create_clock -period 20.000 -name clk [get_ports clk]
   set proj_dir [get_property DIRECTORY [current_project]]
   read_saif "$proj_dir/switching_activity.saif"
   report_power -name structural_power_signoff_report
   ```
4. **Verify Timing Closure:** Under *Implemented Design*, click **Report Timing Summary** and confirm that Setup **Worst Negative Slack (WNS)** remains positive (**+6.537 ns**).

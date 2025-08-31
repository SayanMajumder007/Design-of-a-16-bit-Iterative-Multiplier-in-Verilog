# Design-of-a-16-bit-Iterative-Multiplier-in-Verilog
Designed an iterative multiple which multiplies two unsigned integers. The hardware is partitioned into data path and control path. The data path consists of the functional units where all computations are carried out. Whereas, control path implements a FSM which generates control signals in response to various operations carried out by data path.

Flowcharts and Datapath diagram :
[Datapath_Flowchart.pdf](https://github.com/user-attachments/files/22068132/Datapath_Flowchart.pdf)
<img width="952" height="381" alt="Datapath" src="https://github.com/user-attachments/assets/4407fdb6-ee5b-4685-93bf-14d623fc387c" />
[Controlpath_Flowchat.pdf](https://github.com/user-attachments/files/22068135/Controlpath_Flowchat.pdf)

The verilog code is written in iverilog and the waaveforms are shown in gtkwave.

Verilog Codes :

Datapath top module :
module MUL_datapath (eqz, LdA, LdB, LdP, clrP, decB, data_in, clk);
input LdA, LdB, LdP, clrP, decB, clk;
input [15:0] data_in;
output eqz;
wire [15:0] X, Y, Z, Bout, Bus;
assign Bus = data_in;
PIPO1 A (X, Bus, LdA, clk);
PIPO2 P (Y, Z, LdP, clrP, clk);
CNTR B (Bout, Bus, LdB, decB, clk);
ADD AD (Z, X, Y);
EQZ COMP (eqz, Bout);
endmodule

Sun-modules :
module PIPO1 (dout, din, ld, clk);
input [15:0] din;
input ld, clk;
output reg [15:0] dout;
always @(posedge clk)
if (ld) dout <= din;
endmodule

module PIPO2 (dout, din, ld, clr, clk);
input [15:0] din;
input ld, clr, clk;
output reg [15:0] dout;
always @(posedge clk)
if (clr) dout <= 16'b0;
else if (ld) dout <= din;
endmodule

module CNTR (dout, din, ld, dec, clk);
input [15:0] din;
input ld, dec, clk;
output reg [15:0] dout;
always @(posedge clk)
if (ld) dout <= din;
else if (dec) dout <= din - 1;
endmodule

module EQZ (eqz, data);
input [15:0] data;
output eqz;
assign eqz = (data==0);
endmodule

module ADD (out, in1, in2);
input [15:0] in1, in2;
output reg [15:0] out;
always @(*)
out = in1 + in2;
endmodule

Contral Path :

module MUL_controller (LdA, LdB, LdP, clrP, decB, done, clk, eqz, start);
input clk, eqz, start;
output reg LdA, LdB, LdP, clrP, decB, done;
reg [2:0] state;

parameter S0 = 3'b000, S1 = 3'b001, S2 = 3'b010, S3 = 3'b011, S4 = 3'b100;

always @(posedge clk)
 begin
  case (state)
   S0 : if (start) state <= S1;
   S1 : state <= S2;
   S2 : state <= S3;
   S3 : if (eqz) state <= S4;
   S4 : state <= S4;
   default : state <= S0;
  endcase
 end

always @(state)
 begin
  case (state)
   S0 : begin
         #1 done = 0;
            LdA = 0;
            LdB = 0;
            LdP = 0;
            clrP = 0;
            decB = 0;
        end
   S1 : #1 LdA = 1;
   S2 : begin
         #1 LdA = 0;
            LdB = 1;
            clrP = 1;
        end
   S3 : begin
         #1 LdB = 0;
            clrP = 0;
            LdP = 1;
            decB = 1;
        end
   S4 : begin
         #1 done = 1;
            LdA = 0;
            LdB= 0;
            LdP = 0;
            clrP = 0;
            decB = 0;
        end
   default : begin
              #1 LdA = 0;
                 LdB = 0;
                 LdP = 0;
                 clrP = 0;
                 decB = 0;
             end
  endcase
 end
endmodule

Testbench :

module MUL_test;
reg [15:0] data_in;
reg clk, start;
wire done;

MUL_datapath DP (eqz, LdA, LdB, LdP, clrP, decB, data_in, clk);
MUL_controller CON (LdA, LdB, LdP, clrP, decB, done, clk, eqz, start);

initial
 begin
  clk = 1'b0;
  #3 start = 1'b1;
  #500 $finish;
 end

always #5 clk = ~clk;

initial
 begin
  #17 data_in = 17;
  #10 data_in = 5;
 end

initial
 begin
  $monitor ($time, " %d %b", DP.Y, done);
  $dumpfile ("mul.vcd");
  $dumpvars (0, MUL_test);
 end

endmodule

Simulated Waveform :
<img width="1906" height="939" alt="image" src="https://github.com/user-attachments/assets/d9295eee-277d-40e0-858a-8ee1416704d0" />

Simulation Procedure :
1. Save the verilog files as .v file
2. Opend Comand Window (cmd)
3. Go to C>iverilog>bin
4. write : iverilog MODULE module_name.v module_tb.v and press enter
5. write : vvp MODULE and press enter. Simulated output will be shown
6. Invoke gtkwave to see waveforms : gtkwave Module.vcd (need to initialized in testbench)

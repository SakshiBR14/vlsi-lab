# vlsi-lab
sar
 // ADC controller
 module controller(clk,go,valid,result, sample,value,cmp);
 input
 clk; // clock input
 input
 output
 go; // go=1 to perform conversion
 valid; // valid=1 when conversion finished
 output [7:0] result; // 8 bit result output
 output
 sample; // to S&H circuit
 output [7:0] value; // to DAC
 input
 cmp; // from comparitor
 reg
 reg
 reg
 [1:0] state; // current state in state machine
 [7:0] mask; // bit to test in binary search
 [7:0] result; // hold partially converted result
 // state assignment
 parameter
 sWait=0, sSample=1, sConv=2, sDone=3;
 // synchronous design
 always @(posedge clk) begin
 if (!go) state <= sWait; // stop and reset if go=0
 else case (state)
 // choose next state in state machine
 sWait : state <= sSample;
 sSample :
 begin
 state <= sConv;
 // start new conversion so
 // enter convert state next
 mask <= 8'b10000000; // reset mask to MSB only
 result <= 8'b0;
 // clear result
 end
 sConv :
 begin
 // set bit if comparitor indicates input larger than
 // value currently under consideration, else leave bit clear
 if (cmp) result <= result | mask;
 // shift mask to try next bit next time
 mask <= mask>>1;
 // finished once LSB has been done
 if (mask[0]) state <= sDone;
 end
 sDone :;
 endcase
 end
 assign sample = state==sSample; // drive sample and hold
 assign value = result | mask;
 // (result so far) OR (bit to try)
 assign valid = state==sDone;
 endmodule
 // indicate when finished
Example of Verilog simulation testbench
 (Note: This simulation code requires a simulator which supports such features.)
 module testbench();
 // registers to hold inputs to circuit under test, wires for outputs
 reg clk,go;
 wire valid,sample,cmp;
 wire [7:0] result;
 wire [7:0] value;
 // instance controller circuit
 controller c(clk,go,valid,result, sample,value,cmp);
 // generate a clock with period of 20 time units
 always begin
 #10;
 clk=˜clk;
 end
 initial clk=0;
 // simulate analogue circuit with a digital model
 reg [7:0] hold;
 always @(posedge sample) hold = 8'b01000110;
 assign cmp = ( hold >= value);
 // monitor some signals and provide input stimuli
 initial begin
 $monitor($time, " go=%b valid=%b result=%b sample=%b value=%b cmp=%b
 state=%b mask=%b",
 go,valid,result,sample,value,cmp,c.state,c.mask);
 #100; go=0;
 #100; go=1;
 #5000; go=0;
 #5000; go=1;
 #40; go=0;
 #5000;
 $stop;
 end
 endmodule
6t sram cell
M1 Q QB 0 0 NMOS
M2 QB Q 0 0 NMOS
M3 Q WL BLB 0 NMOS
M4 BL WL QB 0 NMOS
M5 N001 QB Q N001 PMOS
M6 N001 Q QB N001 PMOS
Vdd N001 0 0.85
V2 BL 0 PULSE(0 850m 100p 10p 10p 490p 1n)
V3 BLB 0 PULSE(850m 0 100p 10p 10p 490p 1n)
V4 WL 0 PWL(0 850m 2.49n 850m 2.5n 0 3n 0 3.01n 0 3.02n 850m)
.model NMOS NMOS
.model PMOS PMOS
.lib <Library location>
.tran 8n
.backanno
.end

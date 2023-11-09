# iiitb_r2_4bit_bm -> Radix-2 4-Bit Booth's Multiplier     

### Description 
 This project simulates the Radix-2 4-Bit Booth's Multiplier using Verilog HDL. It can be used to multiply two 4-bit binary signed number in a efficient manner with 
 less number of addition operation.
 
## Introduction
Booth's Multiplier is based on Booth's Multiplication Algorithm. It proposes an efficient way for multiplying two signed integers in there 2's complement form such that the number of partial products is reduced which ultimately lead to the reduction of number of addition operation required for generating the final result.


## Application 
Applications of Booth’s Multiplier

•	They can be used as a fundamental unit for MAC block.<br>
•	They can be used in filters like FIR and IIR.<br>
•	They can be used in Processing Element of Systolic Array

## Block Diagram 

![Circuit_crop](https://github.com/Rohitkadam31/iiitb_r2_4bit_bm-Rohit/assets/148602919/def99784-b204-440e-b602-2637a2249183)<br>As shown in the Block Diagram, Q and M are 4-bit registers that stores the multiplier and multiplicand respectively. It also consist of a 4-bit arthematic unit capable of performing both addition and subtractions. A MOD-4 counter is also present in order to keep track of shifting operation. A and Q are 4-bit shift register. The product is produced in 8_bit register P.<br>
## Working
The multiplication operation through Booth’s algorithm is efficient compared to traditional shift and multiplication process because it produces the result with lesser addition operation.

![Flowchart_crop](https://github.com/Rohitkadam31/iiitb_r2_4bit_bm-Rohit/assets/148602919/79d0463c-c8f6-4547-b9ee-3ebf2c93b35d)
<br>
The above figure shows the flowchart of Booth’s Algorithm. At every clock cycle the two bits (Q0 , Q-1) are inspected for determination of operation .The Q0 refers to Q[0]. If the (Q0 , Q-1) are same (1,1) 0r (0,0) then only right shift operation is performed. If the (Q0 , Q-1) are same (1,0) then subtraction (A-M) followed by right shift is performed. If the (Q0 , Q-1) are same (0,1) then addition (A+M) followed by right shift operation is performed. This cycle is repeated as many times as the specified bits of Booth’s Multiplier is given. This selectivity in performing addition /subtraction operation enables it to produce result efficiently.

* Some Youtube links of working:
  
https://youtube.com/shorts/Ee2VT0C-TUA?si=oTUdyXXB6IwUgPz4

https://youtu.be/DIp4GqSCZho?si=V4mQl17c-KJKyJPn



## Synthesis :

The synthesis in the OpenLane flow is carried out by yosys.

* RTL to synthesize : iiitb_r2_4bit_bm.v

* RTL Code :

 
           module iiitb_r2_4bit_bm(
           input clk,
          	input load,
          	input reset,
	          //inputs
           input [3:0] M,
	          input [3:0] Q,
	
          	//outputs
          	output reg [7:0] P

           );
	 
	          reg [3:0] A 		=  4'b0;
         	 reg Q_minus_one 	=  0;
	          reg [3:0] Q_temp 	=  4'b0;
         	 reg [3:0] M_temp 	=  4'b0;
	          reg [2:0] Count 	=  3'd4;
	 
	 
	 
        	 always @ (posedge clk)
        	 begin
	        	if (reset == 1)
		begin
			A 			 =  4'b0;		//reset values
			Q_minus_one  =  0;
			P 			 =  8'b0;
			Q_temp 		 =  4'b0;
			M_temp 		 =  4'b0;
			Count 		 =  3'd4;

		end

		else if (load == 1)
		begin
			Q_temp 		=  Q;
			M_temp 		=  M;
		end

		else if((Q_temp[0] == Q_minus_one ) && (Count > 3'd0))
		begin
			Q_minus_one =  Q_temp[0];
			Q_temp 		=  {A[0],Q_temp[3:1]};				// right shift Q							
			A 			=  {A[3],A[3:1]};					// right shift A	
		    Count 		=  Count - 1'b1;					
		end
		else if((Q_temp[0] == 0 && Q_minus_one == 1)  && (Count > 3'd0))
		begin
			A 			=  A + M_temp;
			Q_minus_one =  Q_temp[0];
			Q_temp 		=  {A[0],Q_temp[3:1]};				// right shift Q
			A 			=  {A[3],A[3:1]};					// right shift A
			Count 		=  Count - 1'b1;
		end
		else if((Q_temp[0] == 1 && Q_minus_one == 0)  && (Count > 3'd0))
		begin
			A 			=  A - M_temp;
			Q_minus_one =  Q_temp[0];
			Q_temp 		=  {A[0],Q_temp[3:1]};				// right shift Q
			A 			=  {A[3],A[3:1]};					// right shift A
			 Count 		=  Count - 1'b1;
		end
		else 
		begin
			Count = 3'b0;
		end
		P = {A, Q_temp};
		
	         end

          endmodule

* process to synthesize iiitb_r2_4bit_bm on yosys :

         rohit@rohit-VirtualBox:~/vsdflow/verilog/iiitb_r2_4bit_bm $ yosys
         yosys> read_verilog iiitb_r2_4bit_bm.v
         yosys> synth -top iiitb_r2_4bit_bm
         yosys> dfflibmap -liberty ../iiitb_r2_4bit_bm/lib/sky130_fd_sc_hd_tt_025C_1v80.lib
         yosys> abc -liberty ../iiitb_r2_4bit_bm/lib/sky130_fd_sc_hd_tt_025C_1v80.lib
         yosys>show

 * cells invoked :
      
![1 (1)](https://github.com/Rohitkadam31/iiitb_r2_4bit_bm-Rohit/assets/148602919/1489c3e3-671c-48ce-b587-b69888e5e8ac)


* RTL Synthesis :
  
![1 (2)](https://github.com/Rohitkadam31/iiitb_r2_4bit_bm-Rohit/assets/148602919/6e4fb193-df5f-4b0c-be2c-cd456e2082d1)

* RTL Simulation (Pre-synthesis) :

       rohit@rohit-VirtualBox:~/vsdflow/verilog/iiitb_r2_4bit_bm $ iverilog iiitb_r2_4bit_bm.v iiitb_r2_4bit_bm_tb.v
  //a.out created
       rohit@rohit-VirtualBox:~/vsdflow/verilog/iiitb_r2_4bit_bm $ ./a.out
   //  we get vcd file
  ![2 (1)](https://github.com/Rohitkadam31/iiitb_r2_4bit_bm-Rohit/assets/148602919/be72276a-4e7f-4ad6-b6c5-9a9fecc9176e)

       rohit@rohit-VirtualBox:~/vsdflow/verilog/iiitb_r2_4bit_bm $ Gtkwave design.vcd

  We get RTL Simulation i.e Pre-synthesis Simulation

  ![2 (2)](https://github.com/Rohitkadam31/iiitb_r2_4bit_bm-Rohit/assets/148602919/2ee1f4ac-2f47-4720-9e18-57ba6fcfa791)

# Gate Level Simulation (GLS)

(post-synthesis)

         rohit@rohit-VirtualBox:~/vsdflow/verilog/iiitb_r2_4bit_bm $ iverikog /home/rohit/vsdflow/iiitb_r2_4bit_bm/verilog_model/primitives.v 
        /home/rohit/vsdflow/iiitb_r2_4bit_bm/verilog_model/sky130_fd_sc_hd.v iiitb_r2_4bit_bm.v iiitb_r2_4bit_bm_tb.v
 //we get a.out file
 //execute it
 
	      rohit@rohit-VirtualBox:~/vsdflow/verilog/iiitb_r2_4bit_bm $ ./a.out

 //we get dumpfile (vcd file) for output
 
![3 (1)](https://github.com/Rohitkadam31/iiitb_r2_4bit_bm-Rohit/assets/148602919/61a55b53-dbcc-4913-af10-bf8942cd2e1c)

// copy that vcd file and give it to gtkwave for simulation
          
	  rohit@rohit-VirtualBox:~/vsdflow/verilog/iiitb_r2_4bit_bm $ gtkwave design.vcd

![3 (2)](https://github.com/Rohitkadam31/iiitb_r2_4bit_bm-Rohit/assets/148602919/4195fa2f-48e8-4ef9-b2b1-2fe31932e04c)

We can see that GLS simulation matching to RTL simulation 
  

  
  

 








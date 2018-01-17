# Pipelined-processor-for-a-simple-RISC-machine

This project is to design and simulate a pipelined processor for a simple RISC machine.

## Instruction set

	ADD		RD, RS	  add registers RD+RS, store the result in RD
	SUB		RD, RS	  subtract registers RD-RS, store the result in RD
	LOAD		RD,[RS]	  load 1 byte from memory at address RS into RD
	STORE           RD,[RS]	  store 1 byte from RD into memory at address RS
	JLEZ		RD,RS     if RS<=0, jump to address in RD
	JALR		RD,RS     save PC+1 into RS, jump to RD
	LUI		RT, IMM	  load 4-bit IMM into the upper four bits of RT
	LLI		RT, IMM	  load 4-bit IMM into the lower four bits of RT
 
There are four 8-bit registers: A, B, C, D and a program counter PC.  All registers hold a signed 2's complement value. 

## Encoding
```
Registers: A=00, B=01, C=10, D=11

ADD		0 0 0 0 RD1 RD0 RS1 RS0	
					example: ADD B,C = 0 0 0 0 0 1 1 0 = 06h
SUB		0 0 0 1 RD1 RD0 RS1 RS0
					example: SUB D,D = 0 0 0 1 1 1 1 1 = 1Fh
LOAD		0 0 1 0 RD1 RD0 RS1 RS0
					example: LOAD C,[B] = 0 0 1 0 1 0 0 1 = 29h
STORE	        0 0 1 1 RD1 RD0 RS1 RS0
					example: STORE C,[B] = 0 0 1 1 1 0 0 1 = 29h
JLEZ		0 1 0 0 RD1 RD0 RS1 RS0
					example: JLEZ A,B = 0 1 0 0 0 0 0 1 = 41h
JALR		0 1 0 1 RD1 RD0 RS1 RS0
					example: JALR C,D = 0 1 0 1 1 0 1 1 = 5bh
LUI		1 0 RT1 RT0 IMM3 IMM2 IMM1 IMM0
					example: LUI C, 9 = 1 0 1 0 1 0 0 1 = A9h
LLI		1 1 RT1 RT0 IMM3 IMM2 IMM1 IMM0
					example: LLI B, 2 = 1 1 0 1 0 0 1 0 = D2h
```
Write a Fibonacci program in machine code using this instruction set.  Memory address FE is the input and holds the index of the Fibonacci number you're looking for.  Address FF is the output.  

## Fibonacci Program without NOPs

```

..... initialize starting values .....
3 	LUI D,0xf	10111111	bf
4 	LLI D,0xe	11111110	fe	;  D=0xfe
5 	LOAD C,[D]	00101011	2b	;  C=mem[FE]
6 LOOP:	LUI D,0x1	10110001	b1
7 	LLI D,0x6	11110110	f6	;  D=0x1e
8 	JLEZ D,C	01001110	4e	; if C <= 0, goto done
..... perform some Fibonacci calculation here .....
15 	JALR D,D	01011111	5f	; goto loop
16 DONE:LUI D,0xf	10111111	bf
17 	LLI D,0xf	11111111	ff	;  C=0xff
18 	STORE A,[D]	00110011	33	; memory[FF] = A
19 	HALT		01100000	60

```
The program should compute 0 1 1 2 3 5 8 and put 8 in memory[FF]



## Design processor Datapath and control unit

Tool required: Computer Simulator - EmuMaker86
http://www.emumaker86.org/
https://github.com/mdblack/simulator

Using the Datapath Builder, modify the datapath so that it can be pipelined. See datapath.xml

Write control unit for the datapath, control should consist of a fetch state, where the instruction is read from memory, a decode state, which uses the opcode to determine which instruction is to be executed, an execute state with many conditional microinstruction lines, and a writeback state. See control.xml

(datapath.xml and control.xml is not public on github, please contact me if question)

Handling Hazards using data forwarding and squashing.

Data forwarding: Compare the RD code in Writeback with the RS, RD codes in Execute and Decode (this can be done with ALUs). If they are the same, substitute the appropriate value with the writeback code.
Add ALUs and muxes to your datapath. Then add forwarding detection codes to the control unit: add another Control Path and Control State.

Squashing: Keep track of whether or not each stage is invalid, or squashed. If a stage is squash don't perform its memory/port writes, writeback, or data forwarding.
If a jump jlez is detected in Writeback, squash the Fetch, Decode, and Execute stages.
If a jalr is detected in Decode, squash the Fetch stages, and update the PC with address in RD.

## Verilog

Write this datapath and control in Verilog. Test runs the code in Icarus Verilog.
Wire for each bus going into a register.
Added counting variable to check number of cycles and instructions.

Result:

Number of cycles (in decimal)= 120

Number of instructions (in decimal)= 108

The result of an ideal pipelining should have close to 
1 Instruction/Cycle. Mostly because the machine loses 1 cycle on every branching
by squashing. The program did jump 6 times. Cycles loss from branching is 6.
The program exit the loop with JLEZ and 3 cycles wasted by squashing.
Addition to the first 3 cycles use to fillup the pipeline, 12 cycles is wasted during the program running.

## FPGA board

Rewrite Verilog code in to a Xilinx ISE project to get it working on an FPGA board.
Following instruction to setup project:
http://langster1980.blogspot.com/2014/12/numato-mima-v2-tutorial.html


Left seven segment display shows the state: START, FETCH, DECODE, EXECUTE and WRITEBACK.
Center and right seven segment displays show current fetch instruction in Hex value.
LEDs show current PC number in binary.

Feature:

DP1(8) at off position to use auto stepping with delay cycles.

DP1(8) at on position to enable manual stepping mode.

Press SW6 to step to next cycle.

Press SW5 to reviews mem[FF]in Hex value on display and mem[FE] by LEDs in binary.

Press SW4 to reviews Register D in Hex value on display.

Press SW3 to reviews Register C in Hex value on display.

Press SW2 to reviews Register B in Hex value on display.

Press SW1 to reviews Register A in Hex value on display.

## Tools used
EmuMaker86

Icarus Verilog

Mimas V2 Spartan 6 FPGA Development Board

[Assembler](https://github.com/CHico-Lee/Assembler)


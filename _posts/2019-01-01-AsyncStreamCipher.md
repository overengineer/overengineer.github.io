---
layout: post
title: Async Stream Cipher on FPGA
---

> This article explains FPGA implementation of Self-Synchronizing Stream Cipher with Linear Feedback Shift Register using PicoBlaze Microprocessor. You can access to the source files at [here](https://github.com/overengineer/AsyncStreamCipherLFSR).

# Introduction
For recent years, IoT technologies have become very popular. For 2020, the installed base of Internet of Things devices is forecast to grow to almost 31 billion worldwide.[[1]](https://www.statista.com/statistics/471264/iot-number-of-connected-devices-worldwide/) It is undeniable that security for devices that communicating with a worldwide network is a most. In my study, I experienced to design FPGA implementation of a encryption algorithm that used for streaming continuous data. This project is done as final assignment of [Introduction to Embedded System course](https://web.itu.edu.tr/yalcinmust/ehb326.html).
# Theoretical Background
## Stream Ciphers
[Stream ciphers](http://cacr.uwaterloo.ca/hac/about/chap6.pdf) are described as a class of encryption algorithms that encrypt individual characters streaming continuously, contrast to block ciphers which are encrypting a bunch of characters at once. Stream ciphers tend to be faster and simpler than block cipher and they are useful especially when buffering is limited. Additionally, they are useful in communication when it comes to dealing with errors in signal. 
## Self-Synchronizing Stream Cipher
Self-synchronizing or asynchronous stream ciphers are defined as a subset of stream ciphers which their key-stream depends on the key and a certain number of previous cipher-text characters.

## Linear Feedback Shift Register
[Linear feedback shift registers](http://www.eng.auburn.edu/~strouce/class/elec6250/LFSRs.pdf) are a kind of shift register that have favorable statistical properties when producing randomness is desired. They are also easily implemented on hardware.

!![lfsr](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/lfsr.png)


<div style="page-break-after: always;"></div>

# Implementation Details
I designed architecture of my system in two parts consisting of software and hardware. PicoBlaze microprocessor is used in order to reading the plain-text from file and encrypting it using the key-stream read from external LFSR hardware. LFSR algorithm designed as a hardware component on the FPGA and implemented as a Verilog module. 

#### Characteristic polinomial:
 ![latex](https://latex.codecogs.com/gif.latex?P(x)=x^8+x^6+x^5+x^4+1)
 
## Architecture

![scheme1](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/scheme1.png)

In my assignment, it was a constraint of the project to use [PicoBlaze (KPCSM3)](https://www.xilinx.com/products/intellectual-property/picoblaze.html). Therefore, I implemented reading, encrypting and sending data as software to be run inside PicoBlaze. LFSR algorithm is requested to be implemented in hardware, so I implemented it as a Verilog module. PicoBlaze reads data from RAM and encrypts it using LFSR hardware, then sends it to the output port.


## RTL Schematic

![scheme2](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/scheme2.png)


## PicoBlaze
Instructions in PicoBlaze has a constant cycle which is two clock periods. State of the software running inside the processor can be determined outside by counting clock periods or by looking at read_strobe and write_strobe outputs. I have chosen the second method and designed a finite state machine to be implemented as hardware in the top module. I utilized write_strobe and read_strobe outputs for synchronizing the hardware and the software.




## Pseudocode
### Algorithm for Stream Cipher
```python
def stream_cipher(z):
  g = z ^ k
  c = g ^ m
  yield c
```
	  
	  

### Algorithm for LFSR
```python
def lfsr(c):
  f = z | c
  msb = f[0]^f[2]^f[3]^f[4]
  z = {msb, f >> 1}
  yield z
```



<div style="page-break-after: always;"></div>

## Flowchart


![flowchart](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/lfsrflowchart.png)


## Algorithmic State Machine

# Source Codes
	
## Assembly Code
```
NAMEREG	s0,	PUBLIC_KEY
NAMEREG	s1,	MESSAGE
NAMEREG	s2,	CIPHER
NAMEREG	s5,	ram_addr

CONSTANT PUBLIC_KEY_ADDR,	00
CONSTANT DATA_START_ADDR,	01
CONSTANT DATA_END_ADDR,		FF
CONSTANT LFSR_ADDR,			00
CONSTANT PRIVATE_KEY,		54

start:
	INPUT		PUBLIC_KEY,	PUBLIC_KEY_ADDR
	LOAD		ram_addr, DATA_START_ADDR
	OUTPUT  	PUBLIC_KEY, LFSR_ADDR
	JUMP     	first	

loop:
	INPUT		MESSAGE, (ram_addr)
	INPUT		CIPHER,	LFSR_ADDR
	XOR		CIPHER,	PRIVATE_KEY
	XOR		CIPHER, MESSAGE
	OUTPUT	CIPHER, LFSR_ADDR
first:
	;COMPARE	ram_addr, DATA_END_ADDR
	;JUMP		Z, finish
	ADD		ram_addr, 01
	JUMP		loop

finish:
	OUTPUT	PUBLIC_KEY, LFSR_ADDR ; just to signal end of the process
	JUMP finish            ; halt
```
	
<div style="page-break-after: always;"></div>

## Verilog Codes
### TOP.v

```verilog
module TOP(
		input clk, res, start, conf,
		input [7:0] data_in,
		input [7:0] address_in,
		output reg done, busy,
		output valid,
		output [7:0] data_out
    );
	 
//submodule input registers
reg pico_res;

//submodule IO wires
wire ram_wren, ram_res;
wire [7:0] ram_dataout, ram_datain, ram_address;
wire [7:0] lfsr_out, lfsr_in;
wire lfsr_valid,lfsr_res;
wire 	[9:0]	 instr_address ;
wire 	[17:0] instruction ;
wire 	[7:0]	 port_id ;
wire 	write_strobe, read_strobe, interrupt_ack, interrupt ;
wire [7:0] in_port, out_port ;


//submodule instantiations
LFSR lfsr_inst (
    .clk(clk), 
    .res(lfsr_res), 
    .valid(lfsr_valid),  
    .seed(lfsr_in), 
    .keystream(lfsr_out)
    );
	 
RAM  #(.DATA_WIDTH(8),.ADDRESS_WIDTH(8)) ram_inst (
    .clk(clk), 
    .res(ram_res), 
    .wren(ram_wren), 
    .address(ram_address), 
    .datain(ram_datain),
	 .dataout(ram_dataout)
    );

kcpsm3 pico_inst (
    .address(instr_address), 
    .instruction(instruction), 
    .port_id(port_id), 
    .write_strobe(write_strobe), 
    .out_port(out_port), 
    .read_strobe(read_strobe), 
    .in_port(in_port), 
    .interrupt(interrupt), 
    .interrupt_ack(interrupt_ack), 
    .reset(pico_res), 
    .clk(clk)
    );
	 
// Instantiate the module
SSSC instr_memory (
    .address(instr_address), 
    .instruction(instruction), 
    .clk(clk)
    );

//state labels
parameter DATA_WRITE 	 = 3'b000;
parameter IDLE		 = 3'b001;
parameter START		 = 3'b011; 
parameter PUBKEY_READ	 = 3'b010; 
//
parameter PUBKEY_STORE	 = 3'b110; 
parameter DATA_READ	 = 3'b111;
parameter KEYSTREAM_READ = 3'b101;
parameter CIPHER_OUT	 = 3'b100;

//state register
reg [7:0] state;
wire last;

//state logic
always @( posedge clk or posedge res ) begin
	if ( res ) begin
		state <= IDLE;
	end else begin
		case(state)
			DATA_WRITE		: if ( ~conf ) state <= IDLE;
			IDLE				: begin
				if ( pico_res ) begin // ensuring to reset picoblaze two clock cycles
					state <= START;
				end else	if ( conf & ~start ) begin
					state <= DATA_WRITE;
				end
			end
			START				: if ( read_strobe  ) state <= PUBKEY_READ;
			PUBKEY_READ		: if ( write_strobe ) state <= CIPHER_OUT;
			PUBKEY_STORE	: if ( read_strobe  ) state <= DATA_READ;
			DATA_READ		: if ( read_strobe ) state <= KEYSTREAM_READ;
			KEYSTREAM_READ : if ( write_strobe  ) state <= CIPHER_OUT;
			CIPHER_OUT : begin
				if ( last ) begin
					state <= IDLE;//BUG
				end else if ( read_strobe ) begin
					state <= DATA_READ;
				end
			end
		endcase
	end
end

//continious assignments
assign interrupt    = 0;
assign ram_datain   = data_in;
assign ram_res	    = 0;
assign last         = ( port_id == 8'hFF );
assign ram_wren     = conf & ~busy;
assign ram_address  = ram_wren ? address_in : port_id;
assign lfsr_in      = out_port;
assign lfsr_valid   = write_strobe;
assign lfsr_res     = ( state == PUBKEY_READ ) & lfsr_valid;
assign data_out     = out_port;
assign valid        = ( state == KEYSTREAM_READ ) & write_strobe;

parameter RAM = 0, LFSR = 1;
reg select_input;
assign in_port = ( select_input == RAM ) ? ram_dataout : lfsr_out;

//IO logic
always @( posedge clk or posedge res ) begin
	if ( res ) begin
		pico_res      <= 0;
		busy		     <= 1;
		done          <= 0;
	end else begin
		case(state)
			IDLE				: begin
				busy	      <= 0;
				done          <= 0;
				pico_res      <= start;
				if ( pico_res ) begin				
					pico_res      <= 0;
					busy          <= 1;
					select_input  <= RAM;	
				end
			end
			START				: begin
				if ( read_strobe ) begin
					pico_res      <= 0;
				end
			end
			DATA_READ		: begin		
					select_input  <= LFSR;	
				if ( read_strobe ) begin
					done          <= last;
				end
			end
			KEYSTREAM_READ : begin select_input  <= RAM; end
			default: begin end
		endcase
	end
end


endmodule

```



### LFSR.v
```verilog
module LFSR(
    input clk,
    input res,
	 input valid,
    input [7:0] seed,
    output [7:0] keystream
    );
	 reg [7:0] state;
	 wire [7:0] next, feed, shifted;
	 wire msb;
	 
	 assign shifted   = state >> 1; 
	 assign next      = {msb, shifted[6:0]};
	 assign feed      = state | seed;
	 assign msb       = feed[0]^feed[2]^feed[3]^feed[4];
	 assign keystream = state;
	 
	 always @(posedge clk, posedge res) begin
		if (res) begin
			state = seed;
		end else if ( valid ) begin
			state = next;	
		end
	 end
endmodule

```
## Explanation

Top module instiantiates KCPSM3, RAM and LFSR modules and makes their connections. This module contains a finite state machine that changes connections of its submodules. States of the finite state machine inside the top module are changed according to binary outputs of the PicoBlaze instance write_stobe and read_strobe. LFSP module implements linear feedback shift register as keystream generator. It gets feedback from cipher-text output of the PicoBlaze instance. LFSR updates its value when write_strobe output of KCPSM3 becomes 1. PicoBlaze reads data from both RAM and LFSR in different states. Therefore, finite state machine connects different wires to the input port at different states. DATA_WRITE state is added to be used when RAM content is intended to be written from outside.
# Results

## Simulation Waveforms

![wave1](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/wave1.png)

![wave2](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/wave2.png)


## Implementation Reports


![util](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/util.png)

# Conclusion

This project was a good experience for designing an embedded system and making hardware and software cooperate. This porject gave me a wide perspactive when looking at system design problems. I would like to thank Prof.Dr. Müştak Erhan Yalçın for his efforts.


---
layout: post
title: Async Stream Cipher on FPGA
---

> Implementation of Self-Synchronous Stream Cipher with LFSR on FPGA using PicoBlaze Microprocessor

# Introduction
For recent years IoT technologies have become very popular. For 2020, the installed base of Internet of Things devices is forecast to grow to almost 31 billion worldwide.[[1]](https://www.statista.com/statistics/471264/iot-number-of-connected-devices-worldwide/) It is undeniable security for devices that communicating with a worldwide network is a most. In my study, I experienced to design kind of a encryption algorithm that used for streaming continuous data. This project is done as final assignment of [Introduction to Embedded System course](https://web.itu.edu.tr/yalcinmust/ehb326.html).
# Theoretical Background
## Stream Ciphers
[Stream ciphers](http://cacr.uwaterloo.ca/hac/about/chap6.pdf) are described as a class of encryption algorithms that encrypt individual characters streaming continuously, contrast to block ciphers which are encrypting a bunch of characters at once. Stream ciphers tend to faster and simpler than block cipher and they are useful especially when buffering is limited. Additionally, they are useful in communication when dealing with errors in signal. 
## Self-Synchronizing Stream Cipher
Self-synchronizing or asynchronous stream ciphers are defined as a subset of stream ciphers which their key-stream depends on the key and a certain number of previous cipher-text characters.

![selfstream](https://i.imgur.com/Y5n2ygu.png?1)

## Linear Feedback Shift Register
[Linear feedback shift registers](http://www.eng.auburn.edu/~strouce/class/elec6250/LFSRs.pdf) are a kind of shift register that have favorable statistical properties when producing randomness is desired. They are also easily implemented on hardware.

![lfsr](https://i.imgur.com/jQ2xJOF.png)

<div style="page-break-after: always;"></div>

# Implementation Details
I designed architecture of my system in two parts consisting of software and hardware. PicoBlaze microprocessor is used in order to reading the plain-text from file and encrypting it using the  key-stream read from external LFSR hardware. LFSR algorithm designed as a hardware component on the FPGA and implemented as a Verilog module. 

#### Characteristic polinomial:
 <img src="https://latex.codecogs.com/gif.latex?P(x)=x^8+x^6+x^5+x^4+1" /> 
 
## Architecture
In my assignment, it was constraint of the project to use [PicoBlaze (KPCSM3)](https://www.xilinx.com/products/intellectual-property/picoblaze.html). Therefore, I implemented reading, encrypting and sending data as software to be run inside PicoBlaze. LFSR algorithm is requested to be implemented in hardware, so I implemented it as a Verilog module. PicoBlaze reads data from RAM and encrypts it using LFSR hardware, then sends it to the output port.

## PicoBlaze
Instructions in PicoBlaze has a constant cycle which is two clock periods. State of the software running inside the processor can be determined outside by counting clock periods or by looking at read_strobe and write_strobe outputs. I have chosen the second method and designed a finite state machine to be implemented as hardware in the top module. I utilized write_strobe and read_strobe outputs to synchronizing the hardware and the software.




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

## Algorithmic State Machine

## Source Codes
	
### Assembly Code
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
	OUTPUT   PUBLIC_KEY, LFSR_ADDR
	JUMP     first	

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

### Verilog Codes

## Explanation

# Results

## Simulation Waveforms

## RTL Scheme

## Implementation Results

---
layout: post
title: Async Stream Cipher on FPGA
---

# Self-Synchronous Stream Stream with LFSR on FPGA

## Introduction

## Stream Ciphers
Stream ciphers are described as a class of encryption algorithms that encrypt individual characters streaming continuously, contrast to block ciphers which are encrypting a bunch of characters at once. Stream ciphers tend to faster and simpler than block cipher and they are useful especially when buffering is limited. Additionally, they are useful in communication when dealing with errors in signal. 
## Self-Synchronizing Stream Cipher
Self-synchronizing or asynchronous stream ciphers are defined as a subset of stream ciphers which their key-stream depends on the key and a certain number of previous ciphertext characters.
## Linear Feedback Shift Register
Linear feedback shift registers are a kind of shift register that have favorable statistical properties when producing randomness is desired. They are also easily implemented on hardware.
# Implementation Details
I designed architecture of my system in two parts consisting of software and hardware. PicoBlaze microprocessor is used in order to reading the plaintext from file and encrypting it using the  key-stream read from external LFSR hardware. LFSR algorithm designed as a hardware component on the FPGA and implemented as a Verilog module.

#### Characteristic polinomial: 
$$
P(x)=x^8+x^6+x^5+x^4+1
$$

## Architecture

## PicoBlaze
Instructions in PicoBlaze has a constant cycle which is two clock periods. State of the software running inside the processor can be determined outside by counting clock periods or by looking at read_strobe and write_strobe outputs. I have chosen the second method and designed a finite state machine to be implemented as hardware in the top module. I utilized write_strobe and read_strobe outputs to synchronizing the hardware and the software.



## Pseudocode
### Algorithm for Stream Cipher
	def stream_cipher(z):
	  g = z ^ k
	  c = g ^ m
	  yield c	
### Algorithm for LFSR
	def lfsr(c):
	  f = z | c
	  msb = f[0]^f[2]^f[3]^f[4]
	  z = {msb, f >> 1}
	  yield z
## Flowchart
	
## Algorithmic State Machine

## Source Codes
	
### Assembly Code
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
	

### Verilog Codes

## Explanation

# Results

## Simulation Waveforms

## RTL Scheme

## Implementation Results
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTYxNzIzMzUzMV19
-->

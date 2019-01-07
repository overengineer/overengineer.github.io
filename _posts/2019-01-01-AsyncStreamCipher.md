---
layout: post
title: Async Stream Cipher on FPGA
---

> This article explains FPGA implementation of Self-Synchronizing Stream Cipher with Linear Feedback Shift Register using PicoBlaze Microprocessor. You can access to the source files at [here](https://github.com/overengineer/AsyncStreamCipherLFSR).
<div style="page-break-after: always;"></div>


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

<div style="page-break-after: always;"></div>
## RTL Schematic

![scheme2](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/scheme2.png)

<div style="page-break-after: always;"></div>
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

## Algorithmic State Machine

![flowchart](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/lfsrflowchart.png)

<div style="page-break-after: always;"></div>


## Explanation

Top module instiantiates KCPSM3, RAM and LFSR modules and makes their connections. This module contains a finite state machine that changes connections of its submodules. States of the finite state machine inside the top module are changed according to binary outputs of the PicoBlaze instance write_stobe and read_strobe. LFSP module implements linear feedback shift register as keystream generator. It gets feedback from cipher-text output of the PicoBlaze instance. LFSR updates its value when write_strobe output of KCPSM3 becomes 1. PicoBlaze reads data from both RAM and LFSR in different states. Therefore, finite state machine connects different wires to the input port at different states. DATA_WRITE state is added to be used when RAM content is intended to be written from outside.
# Results

## Simulation Waveforms

![wave1](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/wave1.png)

![wave2](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/wave2.png)


## Implementation Reports


![util](https://raw.githubusercontent.com/overengineer/overengineer.github.io/master/images/util.png)
<div style="page-break-after: always;"></div>
# Conclusion

This project was a good experience for designing an embedded system and making hardware and software cooperate. This project gave me a wide perspective when looking at system design problems. I would like to thank Prof.Dr. Müştak Erhan Yalçın for his efforts.


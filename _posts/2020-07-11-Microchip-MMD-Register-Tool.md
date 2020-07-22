---
layout: post
title: "Microchip MMD Register Tool"
author: "Troy Pallay"
date: 2020-07-11 17:23:00 -0400
categories: microchip c software
---

Recently I have been working with Microchip's KSZ9896C Gigabit Ethernet Switch in a board design (datasheet found **[here](https://www.microchip.com/wwwproducts/en/KSZ9896)**). When the switch is handling packets between two computers on two of its ports, it seems to function fine. Good news for my engineering as far as the schematic and impedance control, so that's a relief! However, the board actually has multiple interconnected switches to increase the number of available ports available externally. When I connect to one switch and try to send traffic to a computer on another switch, I get nothing. Well, nothing works perfectly the first time, right?

In my quest for some answers and a solution (Turns out it was an speed auto-negotiation problem between ethernet ports!), my googling took me to this Texas Instruments forum post:
[https://e2e.ti.com/support/interface/f/138/t/821181](https://e2e.ti.com/support/interface/f/138/t/821181).
The resolution to the problem might be applying all patches outlined in the errata document for the KSZ9896C. After reviewing it, I figured that the solutions proposed in the document should be applied to my switches anyways, regardless if they solved my issues.

As of me writing this, the errata document outlines 14 identified issues and offers solutions for most of them (check out that document **[here](http://ww1.microchip.com/downloads/en/DeviceDoc/80000757C.pdf)**). Of these solutions, many involve simply changing some register values. In my case, I am managing the switch and changing these registers over the SPI interface. Performing reads and writes is relatively easy given an address and data. I've included a picture below that shows the format that SPI data should be in for reads and writes. This is given in the datasheet starting on page 53.

![SPI data format](/assets/posts/microchip-mmd-register-tool-01.png)

The difficulty with quite a few of the errata fixes arises from the fact that they deal with MMD registers. These are indirect registers that are not directly accessible, and require a bit more work to read and write to. In fact, the only way to interact with these special registers is through two 'portal registers' that are in the regular address space. page 167 of the KSZ9896C data sheet goes into detail about the process to access the MMD registers, as well as some examples.

![MMD register examples](/assets/posts/microchip-mmd-register-tool-02.png)

In order to actually apply these errata fixes then, it is necessary to translate quite a few mmd register writes into a usable SPI transmittable form. Microchip did not have the data in this format readily on hand, so I have decided to create a quick C program that takes in a file listing the mmd register addresses and data, and spits out a vhdl file holding all the binary data in an array. I do this simply becuase I am communicating with my switches through the use of an onboard FPGA. Hopefully you find this tool useful!

I've made a repo for the program that you can find here:
[https://github.com/troy-pallay/uchip-mmd-register-tool](https://github.com/troy-pallay/uchip-mmd-register-tool)

-Troy

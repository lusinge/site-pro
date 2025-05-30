+++
authors = ["Lucien Gheerbrant"]
title = "CMOS inverter"
date = 2024-08-15
description = "An introductory lab to SPICE software."
[markdown]
numbersections = true
[taxonomies]
tags = ["Semiconductor", "Microelectronic", "Electronic", "SPICE", "S-Edit", "L-Edit"]
[extra]
toc = true
katex = true
+++

## Introduction

After understanding the characteristics of the PMOS and NMOS components,
we will now combine the two to make a CMOS inverter, as in Complementary
MOS. This will allow us to make the simplest digital electronic
component, an inverter. We will also study the layout of a CMOS
inverter.

## CMOS Inverter Operation Analysis

The CMOS schematic is displayed in the figure below. The goal is to take
advantage of the "opposite characteristics" of the nMOS and pMOS
transistors.

If the input voltage is at 0 V, the N-channel transistor is OFF and the
current doesn't go through. The P-channel transistor however is ON and
its current flows freely, as the drain is connected to the source. That
means we get would get the voltage of vdd!\* through in the output, so
2.5 V.


If the input voltage is at 2.5 V, the N-channel transistor is ON and its
current flows freely, and the source get connected to drain. The
P-channel transistor is OFF and its current stops. This time, the output
is the difference between the ground voltage and 0 V, which is 0.

## CMOS Inverter Fabrication Process

To make the CMOS inverter, we need to make an N-channel in a P-type for
the NMOS, and a P-channel for the in a N-type for the PMOS, using
P-wells and N-wells. Poly, meaning polysilicon, is what constitutes the
gate.

## Results

<figure>
    {{
        image(
            url='img/444de6b144356e967521c3b6452f12dcde8202b5.png'
        )
    }}
    <figcaption>
    Figure 1: CMOS inverter schematics, 15/08/2024
    </figcaption>
</figure>

<figure>
    {{
        image(
            url='img/38aa3e7fb993db5e44d9babc3085ad1eecaf2a01.png'
        )
    }}
    <figcaption>
    Figure 2: Pulse signal rising and falling times. 15/08/2024
    </figcaption>
</figure>

<figure>
    {{
        image(
            url='img/c637c9a7fa078516e1ef200a09f6450cbad5a6a5.png'
        )
    }}
    <figcaption>
    Figure 3: Input and output voltage values of a CMOS inverter using a
    pulse source between 0 and 2.5v, 15/08/2024
    </figcaption>
</figure>

<figure>
    {{
        image(
            url='img/518a8a407b66f64fbdcd0d0a0e1f2a4eb836e7df.png'
        )
    }}
    <figcaption>
    Figure 4: Layout of a CMOS inverter, 15/08/2024
    </figcaption>
</figure>

## Conclusions

The simulation of the CMOS respects its inverting characteristics. We
can see the added propagation delay of MOS transistors in the raising
and lowering times.

This inverter is the simplest digital electronic component needed for
modern computing. What we need next is to make a NAND gate to make the
real beginning of Boolean logic through digital electronic.
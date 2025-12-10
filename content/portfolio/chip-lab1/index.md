+++
authors = ["Lucien Gheerbrant"]
title = "Using S-Edit (Schematic Entry Tool) & T-SPICE (Analog Simulation Tool)"
date = 2024-08-09
description = "An introductory lab to SPICE software."
weight = 1
[markdown]
numbersections = true
[taxonomies]
tags = ["Semiconductor", "Microelectronic", "Electronic", "SPICE", "S-Edit"]
[extra]
toc = true
katex = true
+++

This is from an introductory lab to SPICE software. It allows to
design and test circuits.

This lab also focuses on MOS drain current vs. drain-to-source voltage
depending on Vgs values. We also simulate a silicon diode and output its
characteristic.

This lab was really guided and simply introductory. It consisted into
copying the schematics given and give it the specified parameters and
then simulate.

## NMOS

<figure>
    {{
        image(
            url='img/image1.png'
        )
    }}
    <figcaption>
    Figure 1: NMOS
    </figcaption>
</figure>

<figure>
    {{
        image(
            url='img/image2.png'
        )
    }}
    <figcaption>
    Figure 2: NMOS drain current vs. drain-to-source voltage depending on
    Vgs
    </figcaption>
</figure>

## PMOS

<figure>
    {{
        image(
            url='img/image3.png'
        )
    }}
    <figcaption>
    Figure 3: PMOS schematic
    </figcaption>
</figure>

<figure>
    {{
        image(
            url='img/image4.jpeg'
        )
    }}
    <figcaption>
    Figure 4: PMOS drain current vs. drain-to-source voltage depending on
    Vgs
    </figcaption>
</figure>

## Silicon Diode

<figure>
    {{
        image(
            url='img/image5.png'
        )
    }}
    <figcaption>
    Figure 5: Silicon diode schematics
    </figcaption>
</figure>

<figure>
    {{
        image(
            url='img/image6.jpeg'
        )
    }}
    <figcaption>
    Figure 6: Silicon diode characteristic
    </figcaption>
</figure>

## Conclusions

We can see the theoretical functions of the MOS transistors, with the
triode region on the first values of the drain-to-source voltage, and
the linear region.
For the silicon diode, we can see the reversed bias and forward bias as
the current (or voltage) increases.

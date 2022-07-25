# RPM411
Reverse Engineering of the Vaisala RPM411 Pressur Sensor for the RS41 Radiosonde 

For information about the RS41 see [Bajzos website](https://sondehunt.de/language/en/vaisala-rs41), for schematics of it [the RS41 Hardware Repo](https://github.com/bazjo/RS41_Hardware).

The Schematic is in Kicad format, the Parts overlay in svg and can be edited with Inkscape. Some scans in jpeg are provided too.

Pull requests with improvements, translations and bug fixes are welcome!

# Introduction
The sonde is to be divided into three functional blocks, which are seperated by dotted lines in the Schematic.

* [Power Supply](#powersupply)
* [Microcontroller](#microcontroller)
* [Frontend](#frontend)
* [Connector](#connector)

Microcontroller (hidden by a shield with sticker) and Connector are on the Sonde facing side of the PCB, while the Front end usses almost the entire other side, with its power supply weged in the bottom left corner.

Reverse engineering is simpliefied by the fact that the blocks are essentialy smaller Versions of there RS41 counterparts, sharing many parts.

# Schematic
![Schematic](schematic/RPM411.svg?raw=true "Schematic")

# Powersupply
The powersupply (middle right) consists of a Low Dropout Regulator
[TVS70030](http://www.ti.com/lit/ds/symlink/tlv700-q1.pdf) from TI (`U7`), generating 3.3V for the frontend (the MCU is supplyed directly by the Sonde). Pin 4 of the LDO, which is NC according to the datasheet, is decoupled against ground, probably so that pin compatible versions like the [MAX8887](https://datasheets.maximintegrated.com/en/ds/MAX8887-MAX8888.pdf) can be used.

# Microcontroller
The microcontroller is a [STM32F100C8](https://www.st.com/resource/en/datasheet/stm32f100c8.pdf) `U1` from ST in LQFP48 package, the same model as in the Sonde itself, running from an internall clock. Like in the RS41, RC low pass filters are present at many outputs and resistors at many inputs

# Frontend

# Connector
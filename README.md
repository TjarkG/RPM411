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

Reverse engineering is simpliefied by the fact that the blocks are essentialy smaller Versions of there RS41 counterparts, sharing many parts. A significant Part of this describtion is therfore the same as for the RS41 with different Part numbers.

# Schematic
![Schematic](schematic/RPM411.svg?raw=true "Schematic")

## Powersupply
The powersupply (middle right) consists of a Low Dropout Regulator
[TVS70030](http://www.ti.com/lit/ds/symlink/tlv700-q1.pdf) from TI (`U7`), generating 3.3V for the frontend (the MCU is supplyed directly by the Sonde). Pin 4 of the LDO, which is NC according to the datasheet, is decoupled against ground, probably so that pin compatible versions like the [MAX8887](https://datasheets.maximintegrated.com/en/ds/MAX8887-MAX8888.pdf) can be used.

## Microcontroller
The microcontroller is a [STM32F100C8](https://www.st.com/resource/en/datasheet/stm32f100c8.pdf) `U1` from ST in LQFP48 package, the same model as in the Sonde itself, running from an internall clock. Like in the RS41, RC low pass filters are present at many outputs and resistors at many inputs

## Frontend

Electrically, the Frontend is very similar to the RS41's Humindity Messurment, but simplified because there is no temperature Messurment or Heating and all Components are on the PCB. In addition to the Pressure Sensor `P1`, a 15pF reference Capacitor `C33` is available.

The frontend consists of a ring oscillator, whose frequency is varied by variable impedances (e.g. the sensors) inserted into the feedback path with the help of analog switches.

Each ring oscillator is formed by a [74LVC3G04](https://assets.nexperia.com/documents/data-sheet/74LVC3G04.pdf) Triple Inverter `U5`. The three inverters are connected in series. The first and third inverter are feedbacked directly via an RC series element `C26`/`R31`, `R29`/`C23`, the entire ring oscillator via a capacitor `C19`. The ring oscillators can be pulled to +3V by a P-channel MOSFET `Q1` at the first inverter with separate control, which corresponds to ground at the output, to deactivate them.

The feedback path consists of a [74LVC1G32](https://assets.nexperia.com/documents/data-sheet/74LVC1G32.pdf) Or-Gate `U6` used as an Buffer and two resistors `R28` and `R30`, which probably are used for the fine tuning of the resonance frequency. This is followed by three single pole single throw (SPST) switches [TS5A9411](http://www.ti.com/lit/ds/symlink/ts5a9411.pdf) `U2-4`, the NO of which is connected to the output of the buffer and which switch the measuring and reference capacitor either into the feedback network or to ground. `U3` has no connection between COM and the input of the first buffer, where there should normally be a reference capacitor, so this switch does not serve any obvious purpose. However, since it is driven in software like the other two, it can be assumed that the switches have an example-independent non-linear effect on the feedback frequency measured at this switch to compensate for it. Parallel to the switches, fixed feedback is applied via the resistor `R32`.

The measurement output is sent to the MCU via `R8` and `R33`.

## Connector

`J1` connects the Board to the RS41's SPI bus, a clock and two GPIO/CS signals, one of which is shared with the EEPROM, as well as 3V MCU voltage shared with the STM32 on the Sonde and the EEPROM and 3.8V from the boost converter used to supply the Frontend via `U7`. From the six Pins that aren't connected on the RS41, five are used for an SWD interface, the MCU Reset pin and an UART and one remains unused.
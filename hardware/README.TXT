/*
 * Cerberus-Prox HID/Strike Board Files
 *
 * This license covers the schematics, PCB layout,
 * firmware source code and HEX files, readme, and
 * all other files related to the Cerberus-Prox
 * HID/Strike board.
 *
 * Copyright 2008 Andrew Kilpatrick
 *
 * This file is part of Cerberus-Prox.
 *
 * Cerberus-Prox is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 *
 * Cerberus-Prox is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with Cerberus-Prox.  If not, see <http://www.gnu.org/licenses/>.
 */

Cerberus-Prox HID/Strike Control Board
======================================

Introduction
------------
This hardware is a companion for the cerberus-prox Java software that
runs on the client side. The hardware interfaces between a HID
ProxPoint clock and data reader (Model: 6008BGB00 or similar) and
RS-232. It also provides a relay-controlled power output for
controlling a door strike. There are 5 GPIO pins which currently are
not supported by the firmware or protocol.

The HID/strike board should be mounted as close to the HID reader has
possible. Generally this would be on the other side of the door-frame
where the HID reader is installed. Since readers come with fairly
short cables, you might need to extend the included wires on the
reader. Up to a few metres this should not pose a problem.

Circuit Details
---------------
Please review the schematic for information on the hardware. The PCB
is designed for single-sided so that it can be made by hand using DIY
photo etching. Also, only through-hole components have been used to
aid in building by hand. It is suggested to use a small plastic
project enclosure if the system is being installed permanently.

Power Supply:

The power supply should be a DC supply of the same voltage as your
electric door strike. Usually this will be either 12V or 24V. AC can
also be used, but this is not recommended because the voltage seen by
the board may be too high. Whatever voltage you put into the board is
the voltage the strike will see when engaged. This lets you choose the
correct power supply for your strike. The HID/strike board regulates
+5V for its own circuit and for the HID reader. The input voltage
should be between +7 and +24V.

The board with a HID reader attached draws about 90mA at 24V idle, or
about 130mA with the relay switched on. When sizing a supply for your
system, add the power needed by the strike. Multiply by the number of
doors in your system.

HID Reader Wiring:

The HID reader has 10 wires, and all have been implemented on the
HID/strike board. Not all are required for normal operation but it is
suggested that you connect them all for neatness of wiring. If you
need to omit some, it's probably safe to leave RED_LED, HOLD and
CARD_PRESENT lines unconnected as these are not currently used by the
HID/strike board firmware.

Heatsinking U3:

Make sure you use a good heatsink on U3. With 24V going in, under
normal operation U3 will dissipate over 2W. It might get toasty, but
should fine.

Host Wiring
-----------
To connect the HID/strike board to your computer and power, it is
recommended to use CAT5e cable. The recommended colour codes are shown
on the schematic. The power and ground wires use a pair each to avoid
too much voltage drop. The data wires each get their own pair to avoid
crosstalk.

You should be able to get away with up to about 500' of cable between
the host and the HID/strike board. RS-232 is succeptible to noise, so
if you have trouble with long cables try using some RS-485 converters
on each end to balance the signals. For most small installations
RS-232 should work fine and makes it easy to connect to a PC. Each
HID/strike board needs its own RS-232 port. Get a PCI or USB
multi-port RS-232 adapter if you need more ports.

RS-232 Protocol
---------------
The RS-232 protocol for the HID/strike board is designed to be really
simple, and human-readable so that you can test it with a terminal
program. Use 9600 baud, N-8-1 settings on your host with no flow
control enabled. (only TX and RX are used)

All commands and responses are printable characters. Numbers are the
textual representation, not the actual hex byte value. e.h. '0' =
0x30. Responses are also in printable text, and HID data is cooked so
that it can be printed on the screen. (see Java software for decoding
details)

Note that not all commands return responses to the host. On error, the
board doesn't nothing. Sending a newline or carriage return resets the
receiver. HID data is sent as soon as a card is read. Note that the
reader may beep and flash its LED green by itself when a card is
presented. The LED and beep wires appear to OR their inputs with the
internal HID reader logic.


  HID/Strike Board Commands: (send these to the board)

  Commands take the form of: CP\n

	 where: C = command byte
		P = parameter byte
		\n = newline or carriage return (it doesn't matter)

  - 'B' - beep the beeper
    param: the number of seconds to beep (0-9, 0 = off, L = latch on)
    example: "B2\n" - beeps for 2 seconds

  - 'G' - change the HID LED to green
    param: the number of seconds to be green (0-9, 0 = off, L = latch on)
    example: "G1\n" - turn the HID LED green for 1 second

  - 'S' - control the door strike relay
    param: the number of seconds to be green (0-9, 0 = off, L = latch on)
    example: "SL\n" - latch the strike relay on

  - '?' - request device status
    param: send a second '?'
    example: "??\n" - request device status (see response format below)

  HID/Strike Board Responses: (the board sends these to the host)

  - '?' - device status
    param: BGS (B = beep status, G = green LED status, S = strike status)
	       status bytes are '0' = off, '1' = on
    example: "?100\n" - beeper is on, green LED and strike are off
  
  - 'H' - HID card data
    param: CARD_DATA\n (variable number of bytes followed by '\n')
    card data byte format: B'001p dcba'
	 where: p = parity bit
		d = 8s bit
		c = 4s bit
		b = 2s bit
		a = 1s bit

Limitations:

Currently there are 5 GPIO pins supported on the PCB but they are not
implemented in the firmware. This should be fixed in the next version.

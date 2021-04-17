# Geeetech A30T Touch Screen

I have a Geeetech A30T printer with a Smartto board (that is what they call it, it's actually labeled *GTM32\_103\_V1*). It comes with a corresponding firmware from Geeetech.

Unfortunately, the software is a little bit crappy in some aspects, e.g. it does not support some commands or supports only a limited subset. Hence, I want to upgrade to Marlin 2.x.

However, I got the information that the touch screen will not work together with the Marlin firmware. Unfortunately I couldn't get Marlin to work as of now, but if I ever do I want to have the touch screen working as well. I really like to switch filaments or do the leveling via the touch screen. Unfortunately, the firmware of the is closed source, hence, I cannot analyze it by looking at the code. The (unfortunately incomplete) Smartto code that is available from Geeetech via Github does not explain much about the communication.

Hence, I want to analyze the communication between the touch screen board and the control board to be able to use it later on together with a version of Marlin.

## General Information

The screen has it's own board (see picture) and is labeled as *Smartto\_LCD 3.2 VER2.1 FR4 1.6mm 2019-10-31"*. The main processor is an ARM chip labeled *STM32F103 VET6*. This is the same processor as the main board has.

The connector labeled as *J2* seems to be a serial connector as it is connected to the main board via that connector. There are 4 pins and on the main board the pins are labeled *5V*, *RX*, *TX* and *GND* (see picture). The wires of the cable connecting the two ports are running straight. However, the connectors on each side are inverse (see picture).

I bought three USB to TTL adapters via the Internet to be able to connect to the touch screen and to also be able to connect to the control board and the touch screen concurrently to "listen in" on the communication that is happening (see picture). The adapters have an FT232RL chip, but actually I don't know whether this is really required. There are cheaper ones, but for about 10€ for all of them the price was OK for me. I use a break-out board that came with my Raspberry Pi to easily connect the adapter with the boards. The adapters are automatically recognized by my Linux Mint system and devices like */dev/ttyUSB0* are created.

In the Geeetech Smartto repository on Github I saw that a serial port is configured to use 115200 baud. I will use that for testing and see whether that works out OK.

## Approach

My approach to see what the communication pattern between the touch screen and the control board is as follows:

1. Connect the touch screen to Linux and see whether the display works at all. If this works out, check all the screens and see what output they produce.
2. Connect the control board and the touch screen to the Linux system. Forward all messages from one input to the other and also record the communication to see what messages are exchanged.

## Check 1: connect the touch screen to Linux




# Geeetech A30T

I have a Geeetech A30T printer with a Smartto board (that is what they call it, it's actually labeled *GTM32\_103\_V1*). It comes with a corresponding firmware from Geeetech.

Unfortunately, the software is a little bit crappy in some aspects, e.g. it does not support some commands or supports only a limited subset. Hence, I want to upgrade to Marlin 2.x.

This repository is intended for the collection of information about my Geeetech A30T 3D printer. I want to collect all the information for the printer to be able to run it with the Marlin firmware.

## Control board

The board is labeled as `GMT32_103_V1`. There is little to no information currently available on the internet. I didn't find any schematics, however, at least a [picture of where to plug in the cables](GTM32_103_V1%20mainboard.png) is provided by Geeetech.

Schematics are somehow important to 

### Stepper drivers TMC2208

As can be (hardly) seen on the [photo of the board](Board%20with%20connections.JPG), the steppers are driven by TMC2208 chips. I added a [data sheet](TMC2208_Datasheet.pdf) for the steppers. In there, page 8 is the most important one. There I found  information about the pins that have to be connected to the processor for stepping. Mainly, the `ENN`, `DIR` and `STEP` pins are important. The chip can also be controlled via UART, but on this board that is not used.

### Main chip STM32F103VET6

Also visible on the [photo of the board](Board%20with%20connections.JPG) the board has a STM32F103VET6 chip with 100 pins, 512KB flash memory and 64KB SRAM. From what I've seen the chip is quite powerful and has a lot of options. I downloaded a [data sheet](stm32f103ve.pdf) for the chips as well so I could identify the connections on the board. The most interesting pages in this document are page 28 (showing the location of all the pins, PIN 1 is located at the small dot on the chip and the others follow counter-clockwise) and pages 31 through 36 as they describe the different mappings that are possible.

## Marlin 2.x adaption

To make Marlin work ideally the board that the printer uses is already available. This, however, is not true for the A30T. It seems to require a different from all what I've seen in the current [Marlin repo](https://github.com/MarlinFirmware/Marlin).

### How to setup Marlin for the A30T

There are some steps required to make Marlin work with a new printer (as far as I have found out until now):

1. Identify board/chip (mostly written on the board/chip, needed for basic PlatformIO setup)
2. Make a backup (or maybe two?) of your old firmware an ensure you can re-install it
3. Identify the stepper drivers (written on the chip, needed for the pin mapping and config)
4. Identify the boards pin mapping (needed to create a pin mapping)
5. Identify the display you use (if already available, otherwise implement you own)
6. Configure Marlin (use the [example configs](https://github.com/MarlinFirmware/Configurations/tree/import-2.0.x/config/examples))
7. Build Marlin and install the firware

Steps 1. and 3. are already covered above. Step 5. is partly tackled [here](touchscreen/README.md). The results of steps 5. and 6. can be seen in [this repo](https://github.com/TheThomasD/Marlin/tree/geeetech-A30T) if you compare it against the bugfix-2.0.x branch. Also, here is [some additional info](https://marlinfw.org/meta/configuration/) on how to do the configuration. The missing steps 2., 4. and 7. will be explained below.

#### 2. Make a backup and 7. Build Marlin and install Firmware

I basically followed [this](https://github.com/MarlinFirmware/Configurations/blob/0414e840df34a8dbafce07131db4d6cd1ddc9c6a/config/examples/Tronxy/X5SA/HOWTO-INSTALL.md#backup-your-chitu-firmare-optional-but-strongly-recommended) guide to backup the firmware (as the chip is quite similar). I connected a switch to the boot0 pins on the board (they are clearly marked) that allows me to easily switch to the flashing mode (connected = flash enabled). Then, via the STM Cube Programmer, I was able to connect and download the backup via the standard USB port. I had to switch the whole printer on as it seems that the 5V from the USB are not used. There is another jumper on the board that maybe allows to use the 5V via USB, but switching the printer on and off is OK for me, so I never tried to find out. Be careful to not fry your board if you try, though.

The same guid also explains [how to flash a new firmware](https://github.com/MarlinFirmware/Configurations/blob/0414e840df34a8dbafce07131db4d6cd1ddc9c6a/config/examples/Tronxy/X5SA/HOWTO-INSTALL.md#flashing-marlin-firmware-manually-obsolete) as soon as you have it. This is marked as obsolete here, but I haven't tested any other way as it works.

The building process is mainly checking out the branch from github and setting up your Visual Studio Code environment. While the git checkout is not explained (but can easily be found on the internet), [this guide](https://marlinfw.org/docs/basics/install_platformio_vscode.html) explains the basic setup of Marlin with Visual Studio Code.

#### 4. Identify the boards pin mapping

Using a magnifying glass and a simple multimeter I basically followed the path of the electrical components to the pins. Visibly that was mostly impossible as the connections are multi-layered and quite slim. Nevertheless, finding the connections was mostly easy as some of the connections are straigt from a connector to a pin, but in many cases at least a resistor was on the path. There, I checked which resistor was connected to the connector and then used the other side of the resistor as my new starting point. For the stepper drivers I used the pins described above (had to use a needle to reach them, but they are all connected to a resistor nearby which made measuring easier once I found that). Slowly but surely I was able to build the pin mapping. One thing was very trick, though. The 5V connector of the display had to be switched on as it is not connected directly. I had to figure out how the transistors connected to the 5V pin work (identify source, drain and base) and finally find a pin that switched the transistors. Anyway, got it all sorted out. So, here is the mapping. 

|Pin number *as in SPEC*|Pin name|Connector|Description|
---|---|---|---
|1|PE2|FDE0|Filament detector E0|
|2|PE3|E1 ENN|E1 stepper driver not-enable|
|3|PE4|E1 STEP|E1 stepper driver stepping|
|4|PE5|E1 DIR|E1 stepper driver direction|
|5|PE6|FDE1|Filament detector E1|
|7|PC13|E2 ENN|E2 stepper driver not-enable|
|8|PC14|E2 STEP|E2 stepper driver stepping|
|9|PC15|E2 DIR|E2 stepper driver direction|
|15|PC0|T0|Extruder temperature sensor|
|18|PC3|Bed Temp|Bed temperature sensor|
|23|PA0|PB5A|BLtouch servo|
|31|PA6|FANE0|Part blower fan|
|40|PE9|HE0|Extruder heater|
|45|PE14|Bed|Bed heater|
|55|PD8|TX|Display connector TX|
|56|PD9|RX|Display connector RX|
|57|PD10|Z1+|Z0 min endstop *(still, connector reads Z1+)*|
|58|PD11|Y+|Y max endstop *(not used)*|
|59|PD12|5V|Switch on power for display|
|61|PD14|Y ENN|Y axis stepper driver not-enable|
|62|PD15|Y STEP|Y axis stepper driver stepping|
|63|PC6|X+|X max endstop *(not used)*|
|67|PA8|X ENN|X axis stepper driver not-enable|
|70|PA11|X STEP|X axis stepper driver stepping|
|71|PA12|X DIR|X axis stepper driver direction|
|77|PA15|X-|X min endstop|
|84|PD3|Y DIR|Y axis stepper driver direction|
|85|PD4|Y-|Y min endstop|
|86|PD5|Z0 ENN|Z0 axis stepper driver not-enable|
|87|PD6|Z0 STEP|Z0 axis stepper driver stepping|
|88|PD7|Z0 DIR|Z0 axis stepper driver direction|
|89|PB3|Z0-|Bltouch probe Z pin|
|90|PB4|Z1 ENN|Z1 axis stepper driver not-enable|
|91|PB5|Z1 STEP|Z1 axis stepper driver stepping|
|92|PB6|Z1 DIR|Z1 axis stepper driver direction|
|93|PB7|Z1-|Z1 min endstop|
|95|PB8|E0 ENN|E0 stepper driver not-enable|
|96|PD9|E0 STEP|E0 stepper driver stepping|
|97|PE0|E0 DIR|E0 stepper driver direction|
|98|PE1|FDE2|Filament detector E2|

I didn't care to find the SD card connectors as there worked right out of the box after enabling `SDSUPPORT` and `SDIO_SUPPORT`.

One additional note, though: in Marlin some of the ports have different numbers than the spec in the data sheet. Not sure why that is, but at least for pin `PD12` the spec numbers this as 59 and Marlin keeps 60 as the number. This made it somehow confusing as I tried to switch on the pin via GCODE `M42 P59 M1 S1` as a test. Switching from 59 to 60, however, made it work. I found out that mismatch by looking at the output of `M43`. This lists all the used pin numbers an pin names. NB: you need to enable `DIRECT_PIN_CONTROL` and `PINS_DEBUGGING` in `Configuration_adv.h` to use `M42` and `M43`.
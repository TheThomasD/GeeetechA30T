# Geeetech A30T Touch Screen

I got the information that the touch screen will not work together with the Marlin firmware. Although that is true for the time beeing I was able to build [Marlin for my A30T](https://github.com/TheThomasD/Marlin/tree/geeetech-A30T) and was also able to implement my own UI for the touch screen (ongoing, but will also show up on that branch). I really like to switch filaments or do the leveling via the touch screen, that is why I want to use the display. I also would like to avoid buying additional components or modify my complete printer by exchanging the board/display combination completely. 

Unfortunately, the firmware of the touch screen is closed source, hence, I cannot analyze it by looking at the code. The (incomplete) Smartto code that is available from Geeetech via Github ([here](https://github.com/Geeetech3D/Smartto-Eclipse) and [here](https://github.com/Geeetech3D/Smartto-IAR)) does not explain much about the communication.

Hence, I want to analyze the communication between the touch screen board and the control board to be able to use it later on together with a version of Marlin.

## General Information

The screen has it's own board (see [picture](./Touch%20screen.JPG)) and is labeled as *"Smartto\_LCD 3.2 VER2.1 FR4 1.6mm 2019-10-31"*.

The main processor is an ARM chip labeled *"STM32F103 VET6"*. This is the same processor as the main board has. After investing some time in analyzing the main board I'm pretty convinced that it is possible to install a custom firmware. Nevertheless, that would mean developing one basically from scratch, which will maybe be a good idea if I find some spare time (i.e. most likely never). Until then, I'll stick with the original firmware.

The connector labeled as *J2* seems to be a serial connector as it is connected to the main board via that connector. There are 4 pins and on the main board the pins are labeled *5V*, *RX*, *TX* and *GND* (see [picture](./Touch%20screen%20connector%20on%20main%20board.JPG)). The wires of the cable connecting the two ports are running straight. However, the connectors on each side are inverse to each other (see [picture](./Connection%20cable.JPG)).

I bought three USB to TTL adapters (see [picture](./TTL%20FT232RL%20adapter.JPG)) via the Internet to be able to connect to the touch screen and to also be able to connect to the control board and the touch screen concurrently to "listen in" on the communication that is happening. The adapters have an FT232RL chip, but actually I don't know whether this is really required. There are cheaper ones, but for about 10€ for all of them the price was OK for me. I use a break-out board that came with my Raspberry Pi to easily connect the adapter with the boards. The adapters are automatically recognized by my Linux Mint system and devices like */dev/ttyUSB0* are created.

In the Geeetech Smartto repository on Github I saw that a serial port is configured to use 115200 baud. I will use that for testing and see whether that works out OK.

In the following, many codes are postfixed with a star and a number, e.g. `*92`. I assume that these are some kind of check sum as they change based on the content of the message. I have to check on whether I can live without them or have to figure out what they do.

## Approach

My approach to see what the communication pattern between the touch screen and the control board is as follows:

1. Connect the touch screen to Linux and see whether the display works at all. If this works out, check all the buttons on the screens and see what output they produce.
2. Connect the control board and the touch screen to the Linux system via two TTL devices. Forward all messages from one device to the other and also record the communication to see what messages are exchanged.
3. Send messages to the touch screen and the control board and see whether the "check sums" are required or not, and if they are, find out, how they can be derived.

## Check 1: connect the touch screen to Linux

First, I unplug the cable from both boards. I connect the TTL adapter with the control board side of the cable. I connect the cables as they are described on the control board. Then, I use the Linux tool *screen* to connect to the TTL device and see what happens. The command I use is:

`sudo screen /dev/ttyUSB0 115200,cs8`

Then, I connect the other side of the cable with the display. Luckily, the touch screen starts up right away and in the screen program I can see that the display sends the following string initially:

`log Geeetech LCD OK`

The touch screen is usable and for some actions that I do on the touch screen, message are sent from the display. The results are listed in the following table (`N-0` indicates start of a line, I guess these will normally be counted up if the control board responds):

|Menu L1|Menu L2|Menu L3|Button|Command sent|
--- | --- | --- | --- | ---
|**Control**| | | | |
|Control|**Home**| | | |
|Control|Home| |Home all|`N-0 G28*62`|
|Control|Home| |Home X|`N-0 G28 X0*118`|
|Control|Home| |Home Y|`N-0 G28 Y0*119`|
|Control|Home| |Home Z|`N-0 G28 Z0*116`|
|Control|**Move**| | | |
|Control|Move| |Enable|`N-0 M17*56 N-0 G28*62`|
|Control|Move| |Disable|`N-0 M18*55`|
|Control|Move| |X+ (30mm)|`N-0 G91*60 N-0 G1 F6000 X30.00*48 N-0 G90*61`|
|Control|Move| |X- (30mm)|`N-0 G91*60 N-0 G1 F6000 X-30.00*29 N-0 G90*61`|
|Control|Move| |Y+ (30mm)|`N-0 G91*60 N-0 G1 F6000 Y30.00*49 N-0 G90*61`|
|Control|Move| |Y- (30mm)|`N-0 G91*60 N-0 G1 F6000 Y-30.00*28 N-0 G90*61`|
|Control|Move| |Z+ (30mm)|`N-0 G91*60 N-0 G1 F500 Z30.00*1 N-0 G90*61`|
|Control|Move| |Z- (30mm)|`N-0 G91*60 N-0 G1 F500 Z-30.00*44 N-0 G90*61`|
|Control|Move| |switch to 10mm steps|`N-0 L1 S1*76`|
|Control|Move| |switch to 1mm steps|`N-0 L1 S2*79`|
|Control|Move| |switch to 0.5mm steps|`N-0 L1 S3*78`|
|Control|Move| |switch to 0.1mm steps|`N-0 L1 S4*73`|
|Control|Move| |switch to 30mm steps|`N-0 L1 S0*77`|
|Control|**Motion Param**| | | |
|Control|Motion Param| |Steps/mm|`N-0 M2140 S0*122`|
|Control|Motion Param| |Velocity|`N-0 M2140 S1*123`|
|Control|Motion Param| |Acceleration|`N-0 M2140 S2*120`|
|Control|Motion Param| |Jerk|`N-0 M2140 S3*121`|
|Control|Motion Param| |babystep|`N-0 M2140 S5*127`|
|Control|Motion Param| |Double z home offset|`N-0 M2140 S6*124`|
|Control|Motion Param| |Store settings|`N-0 M500*11`|
|Control|Motion Param|**Steps/mm**| | |
|Control|Motion Param|Steps/mm|X-axis up|`N-0 M92 X1.00*82`|
|Control|Motion Param|Steps/mm|X-axis down|nothing, maybe because no original value available|
|Control|Motion Param|Steps/mm|Y-axis up|`N-0 M92 Y1.00*83`|
|Control|Motion Param|Steps/mm|Y-axis down|nothing, maybe because no original value available|
|Control|Motion Param|Steps/mm|Z-axis up|`N-0 M92 Z1.00*80`|
|Control|Motion Param|Steps/mm|Z-axis down|nothing, maybe because no original value available|
|Control|Motion Param|Steps/mm|E0-axis up|`N-0 M92 E1.00*79`|
|Control|Motion Param|Steps/mm|E0-axis down|nothing, maybe because no original value available|
|Control|Motion Param|Steps/mm|Rset|`N-0 M2140 R0*123`|
|Control|Motion Param|**Velocity**| |only display|
|Control|Motion Param|**Acceleration**| |only display|
|Control|Motion Param|**Jerk**| |only display|
|Control|Motion Param|**babystep**| | |
|Control|Motion Param|babystep|babystep up|`N-0 M290 Z0.01*96`|
|Control|Motion Param|babystep|babystep down|`N-0 M290 Z-0.01*77`|
|Control|Motion Param|babystep|Rset|`N-0 M290 Z0*79`|
|Control|Motion Param|**Double z home offset**| | |
|Control|Motion Param|Double z home offset|home-Z0 offset up|`N-0 M2011 P0 S0.01*16`|
|Control|Motion Param|Double z home offset|home-Z0 offset down|`N-0 M2011 P0 S-0.01*61`|
|Control|Motion Param|Double z home offset|home-Z1 offset up|`N-0 M2011 P1 S0.01*17`|
|Control|Motion Param|Double z home offset|home-Z1 offset down|`N-0 M2011 P1 S-0.01*60`|
|Control|Motion Param|Double z home offset|Rset|`N-0 M2011 P0 S0*63 N-0 M2011 P1 S0*62`|
|Control|**Leveling**| | | |
|Control|Leveling|**manual level**| |`N-0 M2107 S0*121`|
|Control|Leveling|manual level|Z up 0.5|`N-0 M2107 S6*127`|
|Control|Leveling|manual level|Z down 0.5|`N-0 M2107 S7*126`|
|Control|Leveling|manual level|Z up 0.05|`N-0 M2107 S10*72`|
|Control|Leveling|manual level|Z down 0.05|`N-0 M2107 S9*112`|
|Control|Leveling|manual level|Pos. 1|`N-0 M2107 S1*120`|
|Control|Leveling|manual level|Pos. 2|`N-0 M2107 S2*123`|
|Control|Leveling|manual level|Pos. 3|`N-0 M2107 S3*122`|
|Control|Leveling|manual level|Pos. 4|`N-0 M2107 S4*125`|
|Control|Leveling|manual level|Pos. 5|`N-0 M2107 S5*124`|
|Control|Leveling|manual level|OK|`N-0 M2107 S8*113`|
|Control|Leveling|manual level|Return|`N-0 G1 F1000 Z10*25 N-0 G28*62 N-0 L101*15`|
|Control|Leveling|**auto-level**| |`N-0 M2120 P5*122`|
|Control|Leveling|auto-level|CAS|`N-0 M2120 P9 S0*53`|
|Control|Leveling|auto-level|3D Touch|`N-0 M2120 P9 S1*52`|
|Control|Leveling|auto-level|Auto-Level on|`N-0 M2120 P0 S1*61`|
|Control|Leveling|auto-level|Auto-Level off|`N-0 M2120 P0 S0*60`|
|Control|Leveling|auto-level|Measure|`N-0 M2120 P7 S0*59 N-0 M2120 P6*121`|
|Control|Leveling|auto-level|3D Touch - Push-pin up|`N-0 M2120 P1 S0*61`|
|Control|Leveling|auto-level|3D Touch - Push-pin down|`N-0 M2120 P1 S1*60`|
|Control|Leveling|auto-level|3D Touch - Alarm release|`N-0 M2120 P1 S2*63`|
|Control|Leveling|auto-level|switch to 0.10mm|`N-0 L2 S2*76`|
|Control|Leveling|auto-level|switch to 0.05mm|`N-0 L2 S3*77`|
|Control|Leveling|auto-level|switch to 10.0mm|`N-0 L2 S0*78`|
|Control|Leveling|auto-level|switch to 1.00mm|`N-0 L2 S1*79`|
|Control|Leveling|auto-level|CAS - up|no visible effect|
|Control|Leveling|auto-level|CAS - down|no visible effect|
|Control|Leveling|auto-level|3D Touch - up 10.0mm|`N-0 M2120 P3 S0*63`|
|Control|Leveling|auto-level|3D Touch - down 10.0mm|`N-0 M2120 P4 S0*56`|
|Control|Leveling|auto-level|3D Touch - up 1.00mm|`N-0 M2120 P3 S1*62`|
|Control|Leveling|auto-level|3D Touch - down 1.00mm|`N-0 M2120 P4 S1*57`|
|Control|Leveling|auto-level|3D Touch - up 0.10mm|`N-0 M2120 P3 S2*61`|
|Control|Leveling|auto-level|3D Touch - down 0.10mm|`N-0 M2120 P4 S2*58`|
|Control|Leveling|auto-level|3D Touch - up 0.05mm|`N-0 M2120 P3 S3*60`|
|Control|Leveling|auto-level|3D Touch - down 0.05mm|`N-0 M2120 P4 S3*59`|
|Control|Leveling|auto-level|Save (value was -0.50)|`N-0 M2120 P1 S2*63 N-0 M2120 P2 S-0.50*56`|
|Control|**Filament**| | | |
|Control|Filament|Extruder 1| |`N-0 M165 A1.0 B0.0 C0.0*67`|
|Control|Filament|Extruder 2| |`N-0 M165 A0.0 B1.0 C0.0*67`|
|Control|Filament|Extruder 1| |`N-0 M165 A0.0 B0.0 C1.0*67`|
|Control|Filament|Hotend on| |`N-0 M104 S200.0*84`|
|Control|Filament|Hotend up| |same as above, but with increased temp value|
|Control|Filament|Hotend down| |same as above, but with decreased temp value|
|Control|Filament|Hotend off| |`N-0 M104 S0.0*86`|
|Control|Filament|Clean on| |`N-0 M2105 S5*126`|
|Control|Filament|Clean off| |`N-0 M2105 S4*127`|
|Control|Filament|Load on (depends on selected extruder, see above)| |`N-0 M165 A1.0 B0.0 C0.0*67 N-0 M2105 S2*121`|
|Control|Filament|Load off| |`N-0 M2105 S4*127`|
|Control|Filament|Unload on (depends on selected extruder, see above)| |`N-0 M165 A1.0 B0.0 C0.0*67 N-0 M2105 S3*120`|
|Control|Filament|Unload off| |`N-0 M2105 S4*127`|
|Control|**Speed**| | | |
|Control|Speed|Speed up (sent value = shown value)| |`N-0 M220 S1*76`|
|Control|Speed|Speed down| |no visible effect|
|Control|Speed|Fan on| |`N-0 M106 P0 S255*8`|
|Control|Speed|Fan off| |`N-0 M106 P0 S0*10`|
|Control|Speed|Fan up| |same as on/off, but with integers representing the requested speed (selected value 0-100 mapped to sent value 0-255)|
|Control|Speed|Fan down| |same as on/off, but with integers representing the requested speed (selected value 0-100 mapped to sent value 0-255)|
|**Printing**| | | |`N-0 M105*10 N-0 M20 LCD*87`|
|Printing| | |all buttons|buttons did not have any effect|
|**Setting**| | | | |
|Setting|**Add-on**| | |`N-0 M2139*55`|
|Setting|Add-on| |Sound on|`N-0 M2139 P0 S1*53`|
|Setting|Add-on| |Sound off|`N-0 M2139 P0 S0*52`|
|Setting|Add-on| |Backlight on|`N-0 M2139 P1 S1*52`|
|Setting|Add-on| |Backlight off|`N-0 M2139 P1 S0*53`|
|Setting|Add-on| |Backlight up (value depends on shown value 0-100)|`N-0 M2139 P1 E51*23`|
|Setting|Add-on| |Backlight down (value depends on shown value 0-100)|`N-0 M2139 P1 E50*22`|
|Setting|Add-on| |Screen lock on|`N-0 M2139 P2 S1*55` Note: screen lock is also stored in touch unit|
|Setting|Add-on| |Screen lock off|`N-0 M2139 P2 S0*54` Note: screen lock is also stored in touch unit|
|Setting|Add-on| |Screen lock PW (value depends on entered value, no leading zeros)|`N-0 M2139 P2 W1234*6` Note: screen lock is also stored in touch unit|
|Setting|Add-on| |Screen lock time (value depends on entered value)|`N-0 M2139 P2 E16*23` Note: screen lock is also stored in touch unit|
|Setting|**Language**| | |no effects|
|Setting|**ScreenCali**| | |no effects|
|Setting|**About**| | |`N-0 M115*11 N-0 N-0 M2134 FW:V1.02.xx*39` NOTE: 2nd and 3rd `N-0` are on the same line!|
|Setting|**Factory Default**| | | |
|Setting|Factory Default| |yes|`N-0 M502*9`|
|Setting|Factory Default| |no|no effect (why should it?)|
|Setting|**Detector**| | | |
|Setting|Detector| |on|`N-0 M2106 P0 S1*57`|
|Setting|Detector| |off|`N-0 M2106 P0 S0*56`|
|**Bar chart button**| | | |`N-0 M27*59`|
|Bar chart button|**Mixer Button**| | | |
|Bar chart button|Mixer Button| |Fixed ratio mixer on|`M2138 S1`|
|Bar chart button|Mixer Button| |Fixed ratio mixer on|`M2138 S0`|
|Bar chart button|Mixer Button| |Change mix *(see mixing ration below, example: 0%, 0%, 100%)*|`M2135 P6553600`|
|Bar chart button|Mixer Button| |Template on|`M2138 S3`|
|Bar chart button|Mixer Button| |Template off|`M2138 S0`|
|Bar chart button|Mixer Button|Template start setting|Height *(height value sent)*|`M2137 C<value>`|
|Bar chart button|Mixer Button|Template start setting|Color *(ratio sent)*|`M2137 A<ratio>`|
|Bar chart button|Mixer Button|Template end setting|Height *(height value sent)*|`M2137 D<value>`|
|Bar chart button|Mixer Button|Template end setting|Color *(ratio sent)*|`M2137 B<ratio>`|
|**Main menu button**| | | |return to top menu|
|**Return button**| | | |return to previous screen|

## Check 2: Listen in on communication between touch screen and control board

I use two TTL devices connected to my Linux system. One connects to the touch screen (as in the section before) and the other one connects to the control board. Hence, I have two devices: `/dev/ttyUSB0` and `/dev/ttyUSB1`. For, it doesn't matter which device is connected to which board as I'm just listening in on the connection and I already know what the messages from the touch screen look like.

I use the tool *"socat"* in Linux to connect the two ports and write two files that contain the communication. The command that I use is

`sudo socat /dev/ttyUSB0,b115200,raw,echo=0 SYSTEM:'tee in.txt |socat - "/dev/ttyUSB1,b115200,raw,echo=0" |tee out.txt'`

This opens the first TTL, sets it to 115200 baud, outputs it to system call that writes the data into the file `in.txt` and forwards the input as well to the second TTL device, setting 115200 baud as well, and captures the output that device into the file `out.txt`. This works out great. The only thing that would be even better is to have a timestamp in the file to see what output from the control board is related to which output from the touch screen.

To do that, I try to change the output of each socket by passing it through the Linux `sed` tool (and some additional magic is required). Hence, my new command is

`sudo socat /dev/ttyUSB0,b115200,raw,echo=0 SYSTEM:'sed -u -E \"s/^(.*)/echo \\\$(date +%T.%N) - \\1/\" | sh | tee in.txt | sed -u \"s/^.*?\-\W//\" | socat - "/dev/ttyUSB1,b115200,raw,echo=0" | tee out-orig.txt | sed -u -E -e \"s/;/#SEMICOLON#/g\" -e \"s/^(.*)/echo \\\$(date +%T.%N) - \\1/\" | sh | tee out.txt | sed -u -e \"s/#SEMICOLON#/;/g\" -e \"s/^.*?\-\W//\"'`

What this does:

1. get the data from the first TTL
2. create a shell command that prepends a timestamp (e.g `echo $(date +%T.%N) - <original message>`)
3. execute the shell command (`sh`) to get the message prepended with the current time (e.g. `15:15:12.123456789 - <original message>`)
4. `tee` the timestamped output into the file `in.txt`
5. remove the timestamp again so that the original message can be sent to the other TTL port
6. send the message to the other port via STDIN with `socat`
7. store the original data in the file `out-orig.txt` for later reference via `tee`
7. replace all semicolons in the output with the text `#SEMICOLON#` (as these would otherwise make trouble with the `sh` tool) and prepare the shell command to prepend the timestamp as above
8. execute `sh` to actually prepend the current timestamp
9. store the timestamped into the file `out.txt`
10. replace the `#SEMICOLON#` markers with actual semicolons again and remove the timestamp
11. data is then sent to the first TTL

The hacky solution with the `sh` command is required as `sed` does not evaluate commands in the replacement part multiple times, so always the same timestamp would be written. However, creating a script to evaluate the timestamp makes it necessary to remove/replace semicolons as they would interfere with the `sh` command. Anyway, the solution works and is good enough to see what is happening when.

### Startup sequence

As already mentioned in the section about the touch screen commands, the first info the touch screen spits out is

`log Geeetech LCD OK`

This seems really to be a log message and I expect that the message is actually logged on the control board if logging is active. Nevertheless, it seems that the message is not required.

When the control board is switched on, it sends out the following messages (I removed the serial number BTW):

```
N-0 L3 PS:0 VL:0 MT:0 FT:0 AL:1 ST:1 WF:0 MR:100 FN: PG:0 TM:0 LA:0 LC:0*5
N-0 L1 X0.000 Y0.000 Z0.000 F0*13
N-0 L2 B:0.0 /0.0 /0 T0:0.0 /0.0 /0 T1:0.0 /0.0 /0 T2:0.0 /0.0 /0 SD:1 F0:0 F2:50 R:100 FR:0*100
N-0 L3 PS:0 VL:0 MT:0 FT:0 AL:1 ST:1 WF:0 MR:100 FN: PG:0 TM:0 LA:0 LC:0*5
N-0 L9 DN:GEEETECH-A30T;DM:A30T;SN:<removed>;FV:V1.xx.03;PV:320.00 x 320.00 x 420.00;HV:V1.6;*6
N-0 L22 MS:0 MR:100 SP:100 EP:100 SH:0.00 EH:0.00*63
N-0 L24 P3 A7.00 B7.00 C0.50 D5.00*126
N-0 L24 P3 A7.00 B7.00 C0.50 D5.00*126
N-0 L24 P3 A7.00 B7.00 C0.50 D5.00*126
N-0 L24 P3 A7.00 B7.00 C0.50 D5.00*126
N-0 L23 SE:0 BE:1 BP:50 CE:0 HE:0 SP:0 ST:16 HC:0 HO:0*23
N-0 L23 SE:0 BE:1 BP:50 CE:0 HE:0 SP:0 ST:16 HC:0 HO:0*23
N-0 L2 B:27.2 /0.0 /0 T0:30.7 /0.0 /0 T1:89.6 /0.0 /0 T2:107.3 /0.0 /0 SD:1 F0:0 F2:50 R:100 FR:0*85
N-0 L18 P26 S1*0
N-0 L21 P0 S0*63
N-0 L3 PS:0 VL:0 MT:0 FT:0 AL:1 ST:1 WF:0 MR:100 FN: PG:0 TM:0 LA:0 LC:0*5
N-0 L21 P0 S0*63
N-0 L21 P0 S0*63
N-0 L3 PS:0 VL:0 MT:0 FT:0 AL:1 ST:1 WF:0 MR:100 FN: PG:0 TM:0 LA:0 LC:0*5
N-0 L21 P0 S0*63
N-0 L21 P0 S0*63
N-0 L2 B:27.3 /0.0 /0 T0:29.0 /0.0 /0 T1:89.4 /0.0 /0 T2:107.4 /0.0 /0 SD:1 F0:0 F2:50 R:100 FR:0*94
N-0 L3 PS:0 VL:0 MT:0 FT:0 AL:1 ST:1 WF:0 MR:100 FN: PG:0 TM:0 LA:0 LC:0*5
N-0 L21 P0 S0*63
[...]
```

The `L21 P0 S0` command is repeated until the touch screen answers with its firmware version. `L1`, `L2` and `L3` seem to be status messages that are sent either regularly (`L3`) or only when something changes. I'll now try to find out what the specific parameters of the messages mean. My findings about the codes and their parameters can be seen in the following two tables.

### Basic data exchange format description

I created this section and also extended the following ones based on the info provided in [this](https://github.com/MarlinFirmware/Marlin/issues/18086#issuecomment-886272862) issue comment. I took the PDF and translated it to [english](SmartoLCD-english.pdf) with Google translate.

|Letter|Definition|
---|---
|**Lnnn**|Commands received by the touch screen|
|**Pnnn**|Command parameters (e.g. file list file name)|
|**Tnnn**|Command parameters, most likely only tool selection|
|**Snnn**|Command parameters (e.g. temperature value)|
|**Xnnn**|X axis coordinate in mm|
|**Ynnn**|Y axis coordinate in mm|
|**Znnn**|Z axis coordinate in mm|
|**Fnnn**|Feedrate in mm/s|
|**Ennn**|Length of extruded material in mm *(I've never ever seen this anywhere being used)*|

Each command sent/received by the control unit or the display is prefixed with the line numer `N-0`, suffixed with a checksum `*nnn` and terminated with `\r\n`. The touch screen receives only `L` nessages, but sends out `G`, `L` and `M` messages. The `G` ones are mostly standard, `L` codes are all proprietary and `M` messages are mixed. In the latter case, the 3-digit codes seem standard and the proprietary ones seem to use 4-digit codes (look also [here](https://github.com/Geeetech3D/Smartto-Eclipse/blob/1a23e6d19976dcbc82589f8e1b547bf4a21a2fcb/STM32f103r/src/Command.c))

### Sent from control board

Examples can be found [here](Examples.md). According to the section above, the display only receives `L` commands.

|Code|Field|Meaning|Answer|
---|---|---|---
|**L1**| |sent to transmit position of X, Y and Z as well as feedrate|
|L1|X|X position in mm|
|L1|Y|Y position in mm|
|L1|Z|Z position in mm|
|L1|F|Feedrate in mm/s|
|**L2**| |sent to transmit status of temperatures etc. (only on changes)|
|L2|B|Bed temperature (current / target / on/off)|
|L2|T0|Extruder 0 temperature (current / target / on/off)|
|L2|T1|Extruder 1 temperature (current / target /on/off) *(shows strange values, machine has only one extruder)*|
|L2|T2|Extruder 2 temperature (current / target /on/off) *(shows strange values, machine has only one extruder)*|
|L2|SD|0 = SD card present, 1 = no SD card present, 2 = SD card failed *(according to documentation also covers U-disk, but no idea what that is (USB disk?) and I don't need it)*|
|L2|F0|Cooling fan speed (values 0 - 100)|
|L2|F2|Motherboard fan speed *(from my research the fan runs at fixed speed and the only value ever provided is 50)*|
|L2|R|Printin speed (values 10 - 800)|
|L2|FR|Feedrate in mm/s|
|**L3**| |transmit print status|
|L3|PS|print status: 0 = idle, 1 = printing, 2 = paused, 3 = recovery, 4 = finished|
|L3|VL|0 = printed from SD, 1 = printed from U-disk|
|L3|MT|motor tension: 0 = disabled, 1 = enabled|
|L3|FT|filament sensor: 0 = on (one sensor has no filament), 1 = on (all sensors detected filament), 255 = off|
|L3|AL|auto leveling: 0 = off, 1 = on|
|L3|ST|Bed leveling sensor type: 0 = capacitive proximity switch, 1 = BLtouch|
|L3|WF|Wifi exists: 0 = no, 1 = yes|
|L3|MR|Mix ratio: a number between 100 (E0 active), 25600 (E1 active) and 6553600 (E2 active). Calculation: lower 8 bits = percent of E0, next 8 bits = percentage E1, upper 8 bits = percentage E2 (all decimal numbers; summing up to 100)|
|L3|FN|file name|
|L3|PG|print progress (0-100 in %)|
|L3|TM|seconds since start of print|
|L3|LA|active layer of current print|
|L3|LC|layer count of current file|
|**L4**| |Wifi status *(will not implement as I don't have the adapter)*|
|**L5**| |Printing rate *(will not implement, haven't seen this, yet)*|
|**L6**| |Fan status *(will not implement, haven't seen this, yet)*|
|**L7**| |SD file list with multiple lines|
|L7|begin file list:n|start of the file list: 0 = SD card|
|L7|P|0-indexed file list of the selected folder|
|L7|end file list:n|end of the file list: 0 = SD card|
|**L8**| |Upload file to screen *(will not implement, haven't seen this, yet)*|
|**L9**| |sends information about the control board, shown in "about" screen. Parameters are semicolon-separated, not space separated|
|L9|DN|Device name|
|L9|DM|Device model|
|L9|SN|Serial number|
|L9|FV|Firmware version|
|L9|PV|Print volume|
|L9|HV|Hardware version|
|**L10**| |Manual leveling result *(can have more parameters as per documentation, but I've never seen that)*|
|L10|S|Z value|
|**L11**| |Automatic leveling saved result|
|L11|P|Always 0|
|L11|S|Z offset|
|**L12**| |*??? P0 S0?*|
|**L13**| | Disk selection *(never seen, will not be implemented)*|
|**L14**| |Send message to printing progress display|
|**L15**| |Filament sensor switch result *(not used, but L3 is used instead)*|
|**L16**| |Power-off resume functionality *(will not implement)*|
|**L17**| |Reserved for motor movement result *(not used)*|
|**L18**| |Error messages|
|L18|P|Interface version *(only seen version 26 seen so far)*|
|L18|S|Error code: 0 = no filament, 1 = SD card removed, 2 = resume print?, 3 = file not found, 4 = hotend temperature abnormal, 5 = bed temperature abnormal, 6 & 7 = bed temperature abnormal, print job stopped, 8 & 9 = bed temperature abnormal, user should kill job, 10 = could not open file, 11 = 3Dtouch alarm (check and retry), 12 = hotend too cold to change filament, 13 = operation unavailable during print, 14 = terminate PID Autotune? (yes -> send M108 and M27 to board), 15 = wait for user to continue (ok -> send M0 to board) >=16 = empty message box with yes and no option|
|**L19**| |Status for stepper control *(will not implement)*|
|**L19**| |Update firmware *(will not implement as not used)*|
|**L21**| |request display firmware version with `P0 S0`|M2134|
|**L22**| |Status of mixing templates|
|L22|MS|*???*|
|L22|MR|Mix ratio, *see L3 MR*|
|L22|SP|Start percentage|
|L22|EP|End percentage|
|L22|SH|Start height|
|L22|EH|End height|
|**L23**| |add-on status|
|L23|SE|sound enabled: 0 = off, 1 = on|
|L23|BE|backlight setting enabled: 0 = off, 1 = on|
|L23|BP|backlight percent: value 1 - 100|
|L23|CE|code enabled: 0 = disabled, 1 = enabled|
|L23|HE|Heater enabled *(not used)*|
|L23|SP|set password: 4 digit value (no leading zeros)|
|L23|ST|screen lock time: number in seconds (at least two digits)|
|L23|HC|Heater center deviation *(not used)*|
|L23|HO|Heater temperature offset *(not used)*|
|**L24**| |info about settings|
|L24|P0|steps/mm: A = X axis, B = Y axis, C = Z axis, D = extruder E0|
|L24|P1|velocities (mm/s): A = X-VMax, B = Y-VMax, C = Z-VMax, D = E-VMax, E = VMin, F = VTravel|
|L24|P2|accelerations (steps/s2): A = Accel, B = A-Retract, C = X-Max accel, D = Y-Max accel, E = Z-Max accel, F = E-Max accel|
|L24|P3|jerks (mm/s): A = Vx-jerk, B = Vy-jerk, C = Vz-jerk, D = Ve-jerk|
|L24|P5|babystep (mm): A = value|
|L24|P6|double-z home offset: A = home-Z0 offset, B= home-Z1 offset

### Sent from touch screen (non standard G-Code as per [this](https://marlinfw.org/meta/gcode/))

Examples can be found [here](Examples.md).

Some definitions can also be found [in the Geeetech Github project](https://github.com/Geeetech3D/Smartto-Eclipse/blob/1a23e6d19976dcbc82589f8e1b547bf4a21a2fcb/STM32f103r/src/Command.c), however, not all of them are accurate (see e.g. filament sensor on/off).

|Code|Field|Meaning|Expected answer|
---|---|---|---
|**L1**| |set stepping for manual move|
|L1|S|0 = 30mm, 1 = 10mm, 2 = 1mm, 3 = 0.5mm, 4 = 0.1mm|
|**L2**| |set stepping for auto leveling|
|L2|S|0 = 10mm, 1 = 1mm, 2 = 0.1mm, 3 = 0.05mm|
|**L101**| |*??? finished manual leveling?*|
|**M20**| |Get SD file list *(requires a specific answer)*|L7|
|**M27, M2101**| |Request print status *(requires a specific answer)*|L3|
|**M104, M105, M140**| |Get/set temperatures *(requires a specific answer)*|L2|
|**M106**| |Set fan speed *(requires a specific answer)*|L6|
|**M114, G1, G28**| |Get/set positions *(requires a specific answer)*|L1|
|**M115**| |Get firmware info *(requires a specific answer)*|L9|
|**M220**| |Set print rate *(requires a specific answer)*|L5|
|**M2011**| |control double z home offset|
|M2011|P|0 = Z0, 1 = Z1|
|M2011|S|offset value|
|**M2100**| |Trigger LCD firmware upgrade *(will not implement)*|
|**M2103**| |*??? something like "start/stop print"?|
|**M2105**| |filament stuff|
|M2105|S|2 = load, 3 = unload, 4 = all motors off, 5 = cleaning on (cycle motors)|
|**M2106**| |filament sensor|
|M2106|P0|S0 = off, S1 = on|
|**M2107**| |manual leveling|
|M2107|S0|home and start|L10, `M2107 Z:%3.2f`|
|M2107|S1|go to pos 1 (RR)|L10, `M2107 ok` or `M2107 fail`|
|M2107|S2|go to pos 2 (RL)|L10, `M2107 ok` or `M2107 fail`|
|M2107|S3|go to pos 3 (FL)|L10, `M2107 ok` or `M2107 fail`|
|M2107|S4|go to pos 4 (FR)|L10, `M2107 ok` or `M2107 fail`|
|M2107|S5|go to pos 5 (C)|L10, `M2107 ok` or `M2107 fail`|
|M2107|S6|Z up 0.5mm|L10, `M2107 Z:%3.2f`|
|M2107|S7|Z down 0.5mm|L10, `M2107 Z:%3.2f`|
|M2107|S8|save|L10, `M2107 save success`|
|M2107|S9|Z up 0.05mm|L10, `M2107 Z:%3.2f`|
|M2107|S10|Z down 0.05mm|L10, `M2107 Z:%3.2f`|
|M2107|S11|exit|L10|
|**M2111**| |Motor movement *(never seen, will not implement)*|
|**M2111**| |Disk selection instruction *(is this required for SD printing?)*|
|**M2120**| |auto leveling|
|M2120|P0|auto level on/off: S0 = off, S1 = on|L11|
|M2120|P1|3D Touch pin control: S0 = pin up, S1 = pin down, S2 = alarm release|
|M2120|P2|store offset value: S = value|L11|
|M2120|P3|offset up: S0 = 10mm, S1 = 1mm , S2 = 0.1mm, S3 = 0.05mm|L1|
|M2120|P4|offset down: S0 = 10mm, S1 = 1mm , S2 = 0.1mm, S3 = 0.05mm|L1|
|M2120|P5|request Z offset|L11|
|M2120|P6|move nozzle to center|L1|
|M2120|P7|S0 = move probe to center and measure|L1|
|M2120|P9|select probing device: S0 = CAS, S1 = 3D Touch|
|**M2130**| |Power-off resume control *(will not implement)*|
|**M2131**| |Configure Wifi *(will not implement)*|
|**M2132**| |Adjusting stepper control *(will not implement)*|
|**M2133**| |Abnormal situation *(not seen yet, will not implement)*|
|**M2134**| |send firmware version|
|M2134|FW:|the firmware version|
|**M2135**| |set current mixing rate|L22|
|M2135|P|Mixing rate bitfield as decimal *(see MR above)*|
|**M2136**| |define mixing template *(not seen yet, will not be implemented)*|L22|
|**M2137**| |set current mixing template *(I've never used this, will most likely not implement)*|L22|
|M2137|A|Mixing ratio start *(see MR above)*|
|M2137|B|Mixing ratio end *(see MR above)*|
|M2137|C|Mixing start layer number|
|M2137|D|Mixing end layer number|
|**M2138**| |Set mixing mode|L22|
|M2138|S|0 = off, 1 = fixed, 3 = template|
|**M2139**| |add-on stuff|
|M2139|P0|sound settings: S0 = off, S1 = on|
|M2139|P1|backlight settings: S0 = off, S1 = on, E<value 0-100> = set brigthness|
|M2139|P2|screen lock: S0 = off, S1 = on, W<4 digits> = set password, E<2 digits> = set lock time|
|M2139|P3|Extruder temperature deiation *(not used)*|
|**M2140**| |control settings|
|M2140|S|0 = steps/mm, 1 = velocity, 2 = acceleration, 3 = jerk, 4 = *???*, 5 = babysteps, 6 = double z home offset|
|**T0, T1, T2**| |Set tool *(requires a specific answer)*|L22|

## Check 3: Send some commands directly to the touchscreen

To get this to work I had to play around a little bit with different commands as I could not figure out at first why I couldn't communicate with the touchscreen.
Nevertheless, after some fiddeling around I found a way to send commands.
I use the command `minicom` to send commands, however, I currently cannot just type them in but I have to type the commands including the newline in a text editor and copy them into minicom.
I guess my CR/LF stuff is currently not working as expected.
Nevertheless, copying and pasting the commands worked.
One of the things I identified is that I have to send the command *including* the `N-0` part at the start *and* I have to add the `*<number here>` part as well.
Hence, I think I have to figure out what the number at the end is.
I guess it's just a simple checksum that has to be calculated as it is always the same if the command does not change.
I'll look at my examples and try to figure out what the numbers mean.

What I've found so far is that the number after the `*` never goes beyond 127, but it seems any number including 0 can show up there.
I've extracted the following commands from my examples to have a basis for further analysis:

```
N-0 L24 P5 A0.03*0
N-0 L24 P5 A0.12*0
N-0 L24 P5 A0.21*0
N-0 L24 P5 A0.30*0

N-0 L24 P5 A0.02*1
N-0 L24 P5 A0.13*1
N-0 L24 P5 A0.20*1

N-0 L24 P5 A0.01*2
N-0 L24 P5 A0.10*2
N-0 L24 P5 A0.23*2

N-0 L24 P5 A0.00*3
N-0 L24 P5 A0.11*3
N-0 L24 P5 A0.22*3

N-0 L24 P5 A0.07*4
N-0 L24 P5 A0.16*4

N-0 L24 P5 A0.06*5
N-0 L24 P5 A0.17*5
N-0 L24 P5 A0.24*5

N-0 L24 P5 A0.05*6
N-0 L24 P5 A0.14*6

N-0 L24 P5 A0.04*7
N-0 L24 P5 A0.15*7

N-0 L24 P5 A0.09*10
N-0 L24 P5 A0.18*10

N-0 L24 P5 A0.08*11
N-0 L24 P5 A0.19*11

N-0 M2120 P9 S1*52
N-0 M2120 P9 S0*53
N-0 M2120 P7 S0*59
N-0 M2120 P0 S0*60
N-0 M2120 P1 S1*60
N-0 M2120 P0 S1*61
N-0 M2120 P1 S0*61

N-0 M2120 P1 S2*63

N-0 M2107 S10*72

N-0 M2107 S9*112
N-0 M2107 S8*113

N-0 M2107 S1*120
N-0 M2107 S0*121
N-0 M2120 P6*121
N-0 M2107 S3*122
N-0 M2120 P5*122
N-0 M2107 S2*123
N-0 M2107 S4*125
N-0 M2107 S7*126
N-0 M2107 S6*127
```

I ordered the first few sorting by command:

```
N-0 L24 P5 A0.00*3
N-0 L24 P5 A0.01*2
N-0 L24 P5 A0.02*1
N-0 L24 P5 A0.03*0
N-0 L24 P5 A0.04*7
N-0 L24 P5 A0.05*6
N-0 L24 P5 A0.06*5
N-0 L24 P5 A0.07*4
N-0 L24 P5 A0.08*11
N-0 L24 P5 A0.09*10
N-0 L24 P5 A0.10*2
N-0 L24 P5 A0.11*3
N-0 L24 P5 A0.12*0
N-0 L24 P5 A0.13*1
N-0 L24 P5 A0.14*6
N-0 L24 P5 A0.15*7
N-0 L24 P5 A0.16*4
N-0 L24 P5 A0.17*5
N-0 L24 P5 A0.18*10
N-0 L24 P5 A0.19*11
N-0 L24 P5 A0.20*1
N-0 L24 P5 A0.21*0
N-0 L24 P5 A0.22*3
N-0 L24 P5 A0.23*2
N-0 L24 P5 A0.24*5
N-0 L24 P5 A0.25*4
N-0 L24 P5 A0.26*7
N-0 L24 P5 A0.27*6
N-0 L24 P5 A0.28*9
N-0 L24 P5 A0.29*8
N-0 L24 P5 A0.30*0
N-0 L24 P5 A0.31*1
N-0 L24 P5 A0.32*2
N-0 L24 P5 A0.33*3
N-0 L24 P5 A0.34*4
N-0 L24 P5 A0.35*5
N-0 L24 P5 A0.36*6
N-0 L24 P5 A0.37*7
N-0 L24 P5 A0.38*8
N-0 L24 P5 A0.39*9
N-0 L24 P5 A0.40*7
```

I also looked for the shortest entries I could find:

```
N-0 G28*62
N-0 M27*59
```

Points that seems interesting:
* since all numbers are lower than 128, maybe there is a modulo by 128 in there?
* I can see from the sorted commands that although single character codes (in ascii) are increasing, the number at the end can increase or decrease independently
* although the commands in the sorted list keep changing, their number does not go beyond 11

Maybe one idea for further investigation:
Send M117 commands to the printer via the USB port and see how these are passed on to the display?

After some googling I found [this](https://reprap.org/wiki/G-code#Checking) information in the Reprap wiki.
It seems, for g-code the characters are just combined using XOR.
That could explain the strange behavior.
I'll have to figure out the encoding now, try to calculate the checksum based on that and compare it with what I've seen in the examples.

I've tested the described algorithm with the default ascii encoding and using XOR on that:
Original line: `N-0 G28*62`
G-code to build checksum for: `N-0 G28`

Decimal [ascii](https://www.torsten-horn.de/techdocs/ascii.htm) values of the characters:
```
N = 78
- = 45
0 = 48
  = 32 (space)
G = 71
2 = 50
8 = 56
```

I put these decimal numbers into [this](https://toolslick.com/math/bitwise/xor-calculator) nice online XOR calculator and, tadaa, the decimal result ist `62` (as what was sent from the control board).
The same worked out for the `N-0 M27*59` command.
I haven't checked the others, but I'd expect them to work as well.
Hence, to send some commands to the display manually I have to send the commands including the checksum.
Now I also think I can understand why Geeetech prefixed all their lines with `N-0 `.
The Reprap wiki entry linked above states that if you want checksums you need line numbers.
I guess Geeetech wanted to have checksums to ensure correct transmission of commands but didn't want to keep track of line numbers.
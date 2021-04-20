# Geeetech A30T Touch Screen

I have a Geeetech A30T printer with a Smartto board (that is what they call it, it's actually labeled *GTM32\_103\_V1*). It comes with a corresponding firmware from Geeetech.

Unfortunately, the software is a little bit crappy in some aspects, e.g. it does not support some commands or supports only a limited subset. Hence, I want to upgrade to Marlin 2.x.

However, I got the information that the touch screen will not work together with the Marlin firmware. Unfortunately I couldn't get Marlin to work as of now, but if I ever do I want to have the touch screen working as well. I really like to switch filaments or do the leveling via the touch screen. Unfortunately, the firmware of the is closed source, hence, I cannot analyze it by looking at the code. The (unfortunately incomplete) Smartto code that is available from Geeetech via Github does not explain much about the communication.

Hence, I want to analyze the communication between the touch screen board and the control board to be able to use it later on together with a version of Marlin.

## General Information

The screen has it's own board (see [picture](./Touch%20screen.JPG)) and is labeled as *"Smartto\_LCD 3.2 VER2.1 FR4 1.6mm 2019-10-31"*.

The main processor is an ARM chip labeled *"STM32F103 VET6"*. This is the same processor as the main board has.

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
|Control|Move| |Enable|`N-0 M18*55`|
|Control|Move| |Disable|`N-0 M17*56 N-0 G28*62`|
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
|**Bar chart button**| | | |`N-0 M105*10 N-0 M27*59`|
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

#### Sent from control board

Examples can be found [here](Examples.md).

|Code|Field|Meaning|Answer|
---|---|---|---
|**L1**| |sent to transmit position of X, Y and Z (and feed rate?)
|L1|X|X position|
|L1|Y|Y position|
|L1|Z|Z position|
|L1|F|*something like feed rate, e.g. is 1 if unloading, but also one if loading*|
|**L2**| |sent to transmit status of temperatures etc. (only on changes)|
|L2|B|Bed temperature (current / target / on/off)|
|L2|T0|Extruder 0 temperature (current / target / on/off)|
|L2|T1|*??? Extruder 1 temperature (current / target / on/off) ??? strange values and machine has only one temp sensor for the hotend*|
|L2|T2|*??? Extruder 2 temperature (current / target / on/off) ??? strange values and machine has only one temp sensor for the hotend*|
|L2|SD|1 = no SD card present, 0 = SD card present|
|L2|F0|Cooling fan speed (values 0 - 100)|
|L2|F2|*???*|
|L2|R|Speed (values 10 - 800)|
|L2|FR|*???*|
|**L3**| |regularly sent to transmit status (of what?)|
|L3|PS|print status: 0 = not printing, 1 = printing, 2 = pause, 3 = *???*, 4 = done|
|L3|VL|*???*|
|L3|MT|motor tension: 0 = disabled, 1 = enabled|
|L3|FT|filament sensor: 0 = on, 255 = off|
|L3|AL|auto leveling: 0 = off, 1 = on|
|L3|ST|*???*|
|L3|WF|*???*|
|L3|MR|Mix ratio: seems to be a number between 100 (E0 active), 25600 (E1 active) and 6553600 (E2 active)|
|L3|FN|file name|
|L3|PG|print progress (0-100 in %)|
|L3|TM|seconds since start of print|
|L3|LA|active layer of current print|
|L3|LC|layer count of current file|
|**L7**| |SD file list|
|L7|begin file list|0 - start of the file list|
|L7|P|0-indexed file list of the selected folder|
|L7|end file list|0 - end of the file list|
|**L9**| |sends information about the control board, shown in "about" screen|
|L9|DN|Device name|
|L9|DM|Device model|
|L9|SN|Serial number|
|L9|FV|Firmware version|
|L9|PV|Print volume|
|L9|HV|Hardware version|
|**L12**| |*??? P0 S0?*|
|**L14**| |*??? seems to send messages to the touch screen*|
|**L18**| |*??? status messages?*|
|L18|P26|SD card status: S0 = SD card in, S1 = SD card out|
|**L21**| |sent with values `P0 S0` until display firmware version from touch screen could be retrieved, not sent afterwards|M2134|
|L21|P|*???*|
|L21|S|*???*|
|**L22**| |Push message for status values|
|L22|MS|*???*|
|L22|MR|Mix ratio, *see L3 MR*|
|L22|SP|*???*|
|L22|EP|*???*|
|L22|SH|*???*|
|L22|EH|*???*|
|**L23**| |add-on status|
|L23|SE|sound enabled: 0 = off, 1 = on|
|L23|BE|backlight setting enabled: 0 = off, 1 = on|
|L23|BP|backlight percent: value 1 - 100|
|L23|CE|code enabled: 0 = disabled, 1 = enabled|
|L23|HE|*???*|
|L23|SP|set password: 4 digit value (no leading zeros)|
|L23|ST|screen lock time: number in seconds (at least two digits)|
|L23|HC|*???*|
|L23|HO|*???*|
|**L24**| |info about settings|
|L24|P0|steps/mm: A = X axis, B = Y axis, C = Z axis, D = extruder E0|
|L24|P1|velocities (mm/s): A = X-VMax, B = Y-VMax, C = Z-VMax, D = E-VMax, E = VMin, F = VTravel|
|L24|P2|accelerations (steps/s2): A = Accel, B = A-Retract, C = X-Max accel, D = Y-Max accel, E = Z-Max accel, F = E-Max accel|
|L24|P3|jerks (mm/s): A = Vx-jerk, B = Vy-jerk, C = Vz-jerk, D = Ve-jerk|
|L24|P5|babystep (mm): A = value|
|L24|P6|double-z home offset: A = home-Z0 offset, B= home-Z1 offset

#### Sent from touch screen (non standard G-Code as per [this](https://marlinfw.org/meta/gcode/))

Examples can be found [here](Examples.md).

|Code|Field|Meaning|Answer|
---|---|---|---
|**L1**| |set stepping for move|
|L1|S|0 = 30mm, 1 = 10mm, 2 = 1mm, 3 = 0.5mm, 4 = 0.1mm|
|**L2**| |set stepping for auto leveling|
|L2|S|0 = 10mm, 1 = 1mm, 2 = 0.1mm, 3 = 0.05mm|
|**L101**| |*??? finished manual leveling?*|
|**M2011**| |control babysteps|
|M2011|P|0 = Z0, 1 = Z1|
|M2011|S|offset value|
|**M2103**| |*??? something like "start/stop print"?|
|**M2105**| |filament stuff|
|M2105|S|2 = load, 3 = unload, 4 = all motors off, 5 = cleaning on (cycle motors)|
|**M2106**| |filament sensor|
|M2106|P0|S0 = off, S1 = on|
|**M2107**| |manual leveling|
|M2107|S|0 = start, 1 = go to pos 1 (RR), 2 = go to pos 2 (RL), 3 = go to pos 3 (FL), 4 = go to pos 4 (FR), 5 = go to pos 5 (C), 6 = Z up 0.5mm, 7 = Z down 0.5mm, 8 = save, 9 = Z down 0.05mm, 10 = Z up 0.05mm|
|**M2120**| |auto leveling|
|M2120|P0|auto level on/off: S0 = off, S1 = on|
|M2120|P1|3D Touch pin control: S0 = pin up, S1 = pin down, S2 = alarm release|
|M2120|P2|store offset value: S = value|
|M2120|P3|offset up: S0 = 10mm, S1 = 1mm , S2 = 0.1mm, S3 = 0.05mm|
|M2120|P4|offset down: S0 = 10mm, S1 = 1mm , S2 = 0.1mm, S3 = 0.05mm|
|M2120|P5|*??? init auto leveling?*|
|M2120|P6|*??? maybe "measure"?*|
|M2120|P7|*??? maybe with S0 "set offset to 0"?*|
|M2120|P9|select probing device: S0 = CAS, S1 = 3D Touch|
|**M2134**| |send firmware version|
|M2134|FW|the firmware version|
|**M2139**| |add-on stuff|
|M2139|P0|sound settings: S0 = off, S1 = on|
|M2139|P1|backlight settings: S0 = off, S1 = on, E<value 0-100> = set brigthness|
|M2139|P2|screen lock: S0 = off, S1 = on, W<4 digits> = set password, E<2 digits> = set lock time|
|**M2140**| |control settings|
|M2140|S|0 = steps/mm, 1 = velocity, 2 = acceleration, 3 = jerk, 4 = *???*, 5 = babysteps, 6 = double z home offset|
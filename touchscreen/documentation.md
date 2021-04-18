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

## Approach

My approach to see what the communication pattern between the touch screen and the control board is as follows:

1. Connect the touch screen to Linux and see whether the display works at all. If this works out, check all the buttons on the screens and see what output they produce.
2. Connect the control board and the touch screen to the Linux system via two TTL devices. Forward all messages from one device to the other and also record the communication to see what messages are exchanged.

## Check 1: connect the touch screen to Linux

First, I unplug the cable from both boards. I connect the TTL adapter with the control board side of the cable. I connect the cables as they are described on the control board. Then, I use the Linux tool *screen* to connect to the TTL device and see what happens. The command I use is:

`sudo screen /dev/ttyUSB0 115200,cs8`

Then, I connect the other side of the cable with the display. Luckily, the touch screen starts up right away and in the screen program I can see that the display sends the following string initially:

`log Geeetech LCD OK`

The touch screen is usable and for some actions that I do on the touch screen, message are sent from the display. The results are listed in the following table (`N-0` indicates start of a line, I guess these will normally be counted up if the control board responds):

|Menu L1|Menu L2|Menu L3|Button|Command sent|
--- | --- | --- | --- | ---
|**Control**|||||
|Control|**Home**||||
|Control|Home||Home all|`N-0 G28*62`|
|Control|Home||Home X|`N-0 G28 X0*118`|
|Control|Home||Home Y|`N-0 G28 Y0*119`|
|Control|Home||Home Z|`N-0 G28 Z0*116`|
|Control|**Move**||||
|Control|Move||Enable|`N-0 M18*55`|
|Control|Move||Disable|`N-0 M17*56 N-0 G28*62`|
|Control|Move||X+ (30mm)|`N-0 G91*60 N-0 G1 F6000 X30.00*48 N-0 G90*61`|
|Control|Move||X- (30mm)|`N-0 G91*60 N-0 G1 F6000 X-30.00*29 N-0 G90*61`|
|Control|Move||Y+ (30mm)|`N-0 G91*60 N-0 G1 F6000 Y30.00*49 N-0 G90*61`|
|Control|Move||Y- (30mm)|`N-0 G91*60 N-0 G1 F6000 Y-30.00*28 N-0 G90*61`|
|Control|Move||Z+ (30mm)|`N-0 G91*60 N-0 G1 F500 Z30.00*1 N-0 G90*61`|
|Control|Move||Z- (30mm)|`N-0 G91*60 N-0 G1 F500 Z-30.00*44 N-0 G90*61`|
|Control|Move||switch to 10mm steps|`N-0 L1 S1*76`|
|Control|Move||switch to 1mm steps|`N-0 L1 S2*79`|
|Control|Move||switch to 0.5mm steps|`N-0 L1 S3*78`|
|Control|Move||switch to 0.1mm steps|`N-0 L1 S4*73`|
|Control|Move||switch to 30mm steps|`N-0 L1 S0*77`|
|Control|**Motion Param**||||
|Control|Motion Param||Steps/mm|`N-0 M2140 S0*122`|
|Control|Motion Param||Velocity|`N-0 M2140 S1*123`|
|Control|Motion Param||Acceleration|`N-0 M2140 S2*120`|
|Control|Motion Param||Jerk|`N-0 M2140 S3*121`|
|Control|Motion Param||babystep|`N-0 M2140 S5*127`|
|Control|Motion Param||Double z home offset|`N-0 M2140 S6*124`|
|Control|Motion Param||Store settings|`N-0 M500*11`|
|Control|Motion Param|**Steps/mm**|||
|Control|Motion Param|Steps/mm|X-axis up|`N-0 M92 X1.00*82`|
|Control|Motion Param|Steps/mm|X-axis down|nothing, maybe because no original value available|
|Control|Motion Param|Steps/mm|Y-axis up|`N-0 M92 Y1.00*83`|
|Control|Motion Param|Steps/mm|Y-axis down|nothing, maybe because no original value available|
|Control|Motion Param|Steps/mm|Z-axis up|`N-0 M92 Z1.00*80`|
|Control|Motion Param|Steps/mm|Z-axis down|nothing, maybe because no original value available|
|Control|Motion Param|Steps/mm|E0-axis up|`N-0 M92 E1.00*79`|
|Control|Motion Param|Steps/mm|E0-axis down|nothing, maybe because no original value available|
|Control|Motion Param|Steps/mm|Rset|`N-0 M2140 R0*123`|
|Control|Motion Param|**Velocity**||only display|
|Control|Motion Param|**Acceleration**||only display|
|Control|Motion Param|**Jerk**||only display|
|Control|Motion Param|**babystep**|||
|Control|Motion Param|babystep|babystep up|`N-0 M290 Z0.01*96`|
|Control|Motion Param|babystep|babystep down|`N-0 M290 Z-0.01*77`|
|Control|Motion Param|babystep|Rset|`N-0 M290 Z0*79`|
|Control|Motion Param|**Double z home offset**|||
|Control|Motion Param|Double z home offset|home-Z0 offset up|`N-0 M2011 P0 S0.01*16`|
|Control|Motion Param|Double z home offset|home-Z0 offset down|`N-0 M2011 P0 S-0.01*61`|
|Control|Motion Param|Double z home offset|home-Z1 offset up|`N-0 M2011 P1 S0.01*17`|
|Control|Motion Param|Double z home offset|home-Z1 offset down|`N-0 M2011 P1 S-0.01*60`|
|Control|Motion Param|Double z home offset|Rset|`N-0 M2011 P0 S0*63 N-0 M2011 P1 S0*62`|
|Control|**Leveling**||||
|Control|Leveling|**manual level**||`N-0 M2107 S0*121`|
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
|Control|Leveling|**auto-level**||`N-0 M2120 P5*122`|
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
|Control|**Filament**||||
|Control|Filament|Extruder 1||`N-0 M165 A1.0 B0.0 C0.0*67`|
|Control|Filament|Extruder 2||`N-0 M165 A0.0 B1.0 C0.0*67`|
|Control|Filament|Extruder 1||`N-0 M165 A0.0 B0.0 C1.0*67`|
|Control|Filament|Hotend on||`N-0 M104 S200.0*84`|
|Control|Filament|Hotend up||same as above, but with increased temp value|
|Control|Filament|Hotend down||same as above, but with decreased temp value|
|Control|Filament|Hotend off||`N-0 M104 S0.0*86`|
|Control|Filament|Clean on||`N-0 M2105 S5*126`|
|Control|Filament|Clean off||`N-0 M2105 S4*127`|
|Control|Filament|Load on (depends on selected extruder, see above)||`N-0 M165 A1.0 B0.0 C0.0*67 N-0 M2105 S2*121`|
|Control|Filament|Load off||`N-0 M2105 S4*127`|
|Control|Filament|Unload on (depends on selected extruder, see above)||`N-0 M165 A1.0 B0.0 C0.0*67 N-0 M2105 S3*120`|
|Control|Filament|Unload off||`N-0 M2105 S4*127`|
|Control|**Speed**||||
|Control|Speed|Speed up (sent value = shown value)||`N-0 M220 S1*76`|
|Control|Speed|Speed down||no visible effect|
|Control|Speed|Fan on||`N-0 M106 P0 S255*8`|
|Control|Speed|Fan off||`N-0 M106 P0 S0*10`|
|Control|Speed|Fan up||same as on/off, but with integers representing the requested speed (selected value 0-100 mapped to sent value 0-255)|
|Control|Speed|Fan down||same as on/off, but with integers representing the requested speed (selected value 0-100 mapped to sent value 0-255)|
|**Printing**||||`N-0 M105*10 N-0 M20 LCD*87`|
|Printing|||all buttons|buttons did not have any effect|
|**Setting**|||||
|Setting|**Add-on**|||`N-0 M2139*55`|
|Setting|Add-on||Sound on|`N-0 M2139 P0 S1*53`|
|Setting|Add-on||Sound off|`N-0 M2139 P0 S0*52`|
|Setting|Add-on||Backlight on|`N-0 M2139 P1 S1*52`|
|Setting|Add-on||Backlight off|`N-0 M2139 P1 S0*53`|
|Setting|Add-on||Backlight up (value depends on shown value 0-100)|`N-0 M2139 P1 E51*23`|
|Setting|Add-on||Backlight down (value depends on shown value 0-100)|`N-0 M2139 P1 E50*22`|
|Setting|Add-on||Screen lock on|`N-0 M2139 P2 S1*55` Note: screen lock is also stored in touch unit|
|Setting|Add-on||Screen lock off|`N-0 M2139 P2 S0*54` Note: screen lock is also stored in touch unit|
|Setting|Add-on||Screen lock PW (value depends on entered value, no leading zeros)|`N-0 M2139 P2 W1234*6` Note: screen lock is also stored in touch unit|
|Setting|Add-on||Screen lock time (value depends on entered value)|`N-0 M2139 P2 E16*23` Note: screen lock is also stored in touch unit|
|Setting|**Language**|||no effects|
|Setting|**ScreenCali**|||no effects|
|Setting|**About**|||`N-0 M115*11 N-0 N-0 M2134 FW:V1.02.xx*39` NOTE: 2nd and 3rd `N-0` are on the same line!|
|Setting|**Factory Default**||||
|Setting|Factory Default||yes|`N-0 M502*9`|
|Setting|Factory Default||no|no effect (why should it?)|
|Setting|**Detector**||||
|Setting|Detector||on|`N-0 M2106 P0 S1*57`|
|Setting|Detector||off|`N-0 M2106 P0 S0*56`|
|**Bar chart button**||||`N-0 M105*10 N-0 M27*59`|
|**Main menu button**||||return to top menu|
|**Return button**||||return to previous screen|


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

The hacky solution with the `sh` command is required as `sed` does not execute commands in the replacement part multiple times, so always the same timestamp would be written. However, creating a script to evaluate the timestamp makes it necessary to remove/replace semicolons as they would interfere with the `sh` command. Anyway, the solution works and is good enough to see what is happening when.


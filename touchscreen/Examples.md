# Examples for touch screen communication

Here I list some examples of some (IMHO noteworthy) codes I've observed being exchanged between the touch screen and the control board (unordered).

## Touch screen to control board

### M20
```
N-0 M20 LCD*87
N-0 M20 file:SD1:/A30T*5
```

### M23
```
N-0 M23 A30Tcube_colored.gco*126
```

### M2134
```
N-0 M2134 FW:V1.02.xx*84
```

## Control board to touch screen

It seems, the control board also spits out messages without a code (maybe for debugging?). Some examples are listed in the first subsection.

### No code
```
Start Print:SD1:/A30T/A30Tcube_colored.gco

SD printing byte 512/1761599

Stop Print

SD printing byte 0/0
```

### L1
```
N-0 L1 X0.000 Y310.000 Z22.675 F8*51

N-0 L1 X0.000 Y0.000 Z12.675 F8*50

N-0 L1 X198.200 Y160.000 Z12.675 F8*55

N-0 L1 X198.200 Y160.000 Z15.000 F100*61

N-0 L1 X0.000 Y0.000 Z10.000 F20*14
```

### L2
```
N-0 L2 B:20.4 /0.0 /0 T0:21.7 /0.0 /0 T1:88.6 /0.0 /0 T2:108.0 /0.0 /0 SD:0 F0:0 F2:50 R:100 FR:8*80

N-0 L2 B:20.4 /0.0 /0 T0:21.7 /0.0 /0 T1:88.6 /0.0 /0 T2:108.0 /0.0 /0 SD:0 F0:0 F2:50 R:100 FR:37*108

N-0 L2 B:21.4 /0.0 /0 T0:22.9 /0.0 /0 T1:88.4 /0.0 /0 T2:107.8 /0.0 /0 SD:0 F0:0 F2:50 R:97 FR:5*107

N-0 L2 B:21.3 /0.0 /0 T0:27.0 /0.0 /0 T1:89.9 /0.0 /0 T2:107.5 /0.0 /0 SD:0 F0:100 F2:50 R:10 FR:5*111
```

### L3
```
N-0 L3 PS:0 VL:0 MT:0 FT:0 AL:1 ST:1 WF:0 MR:100 FN: PG:0 TM:0 LA:0 LC:0*5

N-0 L3 PS:1 VL:0 MT:0 FT:0 AL:1 ST:1 WF:0 MR:100 FN:A30T/A30Tcube_colored.gco PG:0 TM:0 LA:0 LC:0*92

N-0 L3 PS:2 VL:0 MT:1 FT:0 AL:1 ST:1 WF:0 MR:100 FN:A30T/A30Tcube_colored.gco PG:0 TM:0 LA:0 LC:0*94

N-0 L3 PS:4 VL:0 MT:1 FT:0 AL:1 ST:1 WF:0 MR:100 FN:A30T/A30Tcube_colored.gco PG:100 TM:0 LA:1 LC:0*88

N-0 L3 PS:0 VL:0 MT:1 FT:0 AL:1 ST:1 WF:0 MR:100 FN:A30T/A30Tcube_colored.gco PG:0 TM:0 LA:0 LC:0*92
N-0 L3 PS:0 VL:0 MT:0 FT:0 AL:1 ST:1 WF:0 MR:100 FN:A30T/A30Tcube_colored.gco PG:0 TM:0 LA:0 LC:0*93

N-0 L3 PS:0 VL:0 MT:1 FT:0 AL:0 ST:1 WF:0 MR:25600 FN: PG:0 TM:0 LA:0 LC:0*5
N-0 L3 PS:0 VL:0 MT:1 FT:0 AL:0 ST:1 WF:0 MR:6553600 FN: PG:0 TM:0 LA:0 LC:0*7

N-0 L3 PS:0 VL:0 MT:1 FT:255 AL:0 ST:1 WF:0 MR:100 FN: PG:0 TM:0 LA:0 LC:0*7

```

### L7
```
N-0 L7 begin file list:0*65
N-0 L7 P0 A30T*126
N-0 L7 end file list:0*73

N-0 L7 begin file list:0*65
N-0 L7 P0 Main/A30T*122
N-0 L7 P1 A30Tcube_colored.gco*8
N-0 L7 P2 A30T.GCO*25
N-0 L7 P3 5. Unsliced Modle*16
N-0 L7 P4 4. Troubleshooting Guide*116
N-0 L7 P5 3. Color Mixer*102
N-0 L7 P6 2. Slicing Software and Drivers*46
N-0 L7 P7 1. User Manuals*8
N-0 L7 end file list:0*73
```

### L10
```
N-0 L10 S8.00*91
N-0 L10 S8.50*94
N-0 L10 S9.00*90
N-0 L10 S9.50*95
```

### L11
```
N-0 L11 P0 S-2.67*60
```

### L12
```
N-0 L12 P0 S0*63
```

### L14
```
N-0 L14 Pausing... please try again later.*

N-0 L14  192.168.0.144 3344*48

N-0 L14 file select succeed!:a30tcube_colored.gco
```

### L18
```
N-0 L18 P26 S0*1

N-0 L18 P26 S1*0
```

### L21
```
N-0 L21 P0 S0*63
```

### L22
```
N-0 L22 MS:0 MR:100 SP:100 EP:100 SH:0.00 EH:0.00*63

N-0 L22 MS:0 MR:6553600 SP:100 EP:100 SH:0.00 EH:0.00*61
```

### L23
```
N-0 L23 SE:1 BE:1 BP:50 CE:0 HE:0 SP:0 ST:16 HC:0 HO:0*22

N-0 L23 SE:0 BE:1 BP:50 CE:0 HE:0 SP:0 ST:16 HC:0 HO:0*23

```

### L24
```
N-0 L24 P0 A80.00 B80.00 C400.00 D390.00*115

N-0 L24 P1 A150 B150 C20 D25 E1 F5*126

N-0 L24 P2 A500 B500 C500 D500 E50 F1000*120

N-0 L24 P3 A7.00 B7.00 C0.50 D5.00*126

N-0 L24 P5 A0.00*3

N-0 L24 P6 A0.01 B0.00*125

```

### M2107
```
M2107 Z:8.00

M2107 ok
```
# Construction Note for TinyZ280, UART Bootstrap Configuration
–> bare board photos here <–

### Bill of materials for TinyZ280:
```
  Item     Qty  References                              Value    Module Name
------------------------------------------------------------------------------
     1       2  R32,R33                                    1K          R0805
     2      17  R8,R9,R10,R11,R12,R13,R14,R15,            100          R0805
                R16,R17,R18,R19,R20,R21,R22,
                R23,R25
     3       2  D1,D2                                     LED          R0805
     4       1  R5                                       2.6K          R1206
     5      11  R1,R2,R4,R6,R7,R24,R26,R28,              4.7K          R0805
                R29,R30,R31
     6       1  U1                                       Z280    PLCC68SOCKE
     7       9  C1,C2,C3,C4,C5,C6,C7,C8,C9              0.1uF          C1206
     8       1  C14                                     100pF          C0805
     9       1  U6                                      24MHZ        4DIP300
    10       8  T1,T2,T3,T4,T5,T6,T14,T15               TERM1          JUMP1
    11       7  T7,T8,T9,T10,T11,T12,T13                TERM1       JUMP1-35
    12       1  P42                                    IDE44B         IDE44B
    13       1  P41                                    IDE44T         IDE44T
    14       1  P1                                     SIMM72        SIMM72S
    15       1  U9                                    DS12887       24DIP600
    16       1  P3                                    in-situ         HDR2X5
    17       1  U8                                    MCP130D           TO92
    18       8  W1,W2,W3,W4,W5,W6,W7,W8               RINGPAD          PAD50
    19       4  C10,C11,C12,C13                      10uF-SMT         TC3528
    20       2  U3,U4                                AM29F010       32DIP600
    21       2  U5,U7                                AT24C256        8DIP300
    22       1  U10                                  COM81C17       20DIP300
    23       1  U11                                  DUAL-OSC        8DIP300
    24       1  J1                                  JACK2_5MM      2.5mm_new
    25       2  R3,R27                              SIP8 4.7K           SIP8
    26       1  U2                              EPM7128SQC100    7128SQC100H
    27       2  S1,S2                           SW_PUSHBUTTON         MINI-4
```
### Assembly procedure:

1. Solder U2, EPM7128SQC100, with a fine tip soldering pen. Use plenty of flux.
2. Install R32 and R31. They are 0805 resistors and value can be lowered to 470 ohms for brighter display
3. Install 0805 LED D1 and D2. Observe the polarity marking with respect to the '+' silkscreen mark.
4. Install C10, 10uF filter cap. Follow the polarity marking
5. Install R25, R8-R15 (100 ohm resistors), C14 (100pF), and R24, R26 (4.7K)
6. Flip the pc board over and install all SMT passive components
7. Install the 2 SIP resistors, R3, R27
8. Install the 4 pin sockets for U6 (24MHz full size oscillator)
9. Install 32-pin socket for U3 and U4
10. install the 4-pin jumper block for J3-J6. The hole size is too small for wire-wrap pins, so use dual row WW pin header with 20 mil tails.
11. Install U11, EC300 14.7Mhz dual oscillator.
12. Install a jumper wire from T14 to T15
13. Install P3, Altera programming header.
14. Install the two push button switches for NMI and RESET
15. Install 2.5mm power jack
16. Install voltage supervisor, MCP130D.
17. Install 68-pin socket for U1, observe the polarity marking of the socket.
18. Install the 4-pin serial port connector. Only solder three pins, GND, Rx, Tx. Cut short the pin meant for VCC, solder a wire from '+' terminal on pcb to the pin that was cut short.
19. Wash the board
20. Program Altera EPM7128 first before populating the board
21. Jumper J3-J6 block as J3-J6, J4-J5. This configures U3/U4 as 128Kx8 RAM
22. Power up, current consumption should be around 270mA. Load TinyLoad and Glitchmon and observe proper operations
23. Solder CF adapter.
24. Try cpm22all with CP/M22 distro in CF drive.
25. Do not populate R11 and R15
26. Install 5 bypass capacitors, C4-C8

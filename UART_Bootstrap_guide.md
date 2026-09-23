# Operation Guide for TinyZ280 UART Bootstrap Configuration


![uart](TinyZ280_topview.jpg)
Picture above shows the TinyZ280 board configured for UART bootstrap.

- U3 and U4 are populated with 128K X 8 static RAM and the jumper blocks J3-J6 are strapped as J3-to-J6, J4-to-J5.
- CPLD is programmed with UART_bootstrap_configuration
- Terminal software is configured as 57600 baud, odd parity, 8 bit data, 1 stop bit.

Software required are

- TinyLoad
- Glitchmon
- CPM22all
To run CPM 2.2, the CF disk should be loaded with CPM 2.2 distribution files.

Z280 is in UART bootstrap mode when power is applied or when the RESET button is pressed. The board is expecting a 256-byte serial binary data stream from the terminal. Use the 'send file' function of the terminal software to send 'TinyLoad.bin' to the board. TinyLoad.bin is a binary file, some terminal software requires binary option to be specified when sending TinyLoad.bin. TinyLoad is a 256-byte software; Z280 is configured to DMA the serial stream from UART to memory starting from location 0. After 256 bytes of data is received, Z280 will start program execution at location 0. It will display the following message:
```
TinyLoad 1  <-- version number, likely to change
G xxxx when done
```
TinyLoad has three functions:

1. It clears memory from 0x100 to 0xFFFF to zero.
2. It expects Intel hex file and save it to memory specified by the load address. It will check every record and print a period (.) if the checksum matches or question mark (?) if the checksum does not match. It will output 'U' for unrecognized record format and 'X' for end of record.
3. It recognizes the 'G' command and transfers the control to the 4-byte address follow the 'G' command. Please note: the 4-byte address is not echo back on the terminal, only the 'G' followed by a blank is displayed.

After TinyLoad is running, it can receive one or more hex load files. A small monitor program, Glitchmon.hex , can be loaded. At the end of a successful load operation ('X' at the end of load and no '?' displayed), type 'G 0200' (please note the address '0200' will not echo back) to run Glitchmon. The following sign-on message will be displayed:
```
........................................UX
G Glitch Works Monitor for 8080/8085/Z80 and Compatible
Version 0.1 Copyright (c) 2012 Jonathan Chapman
1/24/18 Modified by Bill Shen for TinyZ280
Built for Zilog Z280>
```
Instruction for Glitchmon is here: https://github.com/chapmajs/glitchworks_monitor/blob/master/README.md

This is a quick summary of the commands:
```
D XXXX YYYY    Dump memory from XXXX to YYYY
E XXXX   Edit memory starting at XXXX (type an X and press enter to exit entry)
G XXXX        GO starting at address XXXX (JMP in, no RET)
I XX        Input from I/O port XX and display as hex
O XX YY        Output to I/O port XX byte YY
```
Another load file for TinyLoad is cpm22all.hex. After loading is completed, type 'G f200' to start the cpm 2.2.
```
Test CP/M TinyZ280
2/1/18

a>
```
Assuming drive B of CF contains CP/M distro, 'dir' command will display the following:
```
a>b:
b>dir
B: XSUB     COM : SUBMIT   COM : STAT     COM : READ     ME
B: PIP      COM : LOAD     COM : ED       COM : DUMP     COM
B: DUMP     ASM : DSKMAINT COM : DISKDEF  LIB : DEBLOCK  ASM
B: DDT      COM : CPM      SYS : BIOS     ASM : ASM      COM
b>
```
The CP/M 2.2 implementation is buggy. There are no CP/M image in the boot track. Warm boot is just cold boot without the sign on message.


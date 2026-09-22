# TinyZ280 Software Build Procedures
This article describes the tools and process of building software for TinyZ280

All software are written in Z80 assembly language. Z280 instructions are used where needed and are hand assembled into in-line constants. Two Windows-based Z80 assemblers were used to build the software:

Zilog ZDS v3.68 available for free download at Zilog.com

ZMAC–Z80 Macro Cross Assembler is available for download at http://48k.ca/zmac.html ZMAC is needed to produce relocatable file format (.rel) which is required to build the CP/M 3 system files.

## CFMon and CFMonLdr build procedure
CFMon is a small file loader who only function is to load ZZMon from its predetermined location in CF (LBA sector 0xF8-0xFF) into memory at 0x400-0xFFF and jump into 0x400. CFMon is located in the boot sector of a CF (LBA sector 0). At reset or power-up, the CPLD state machine initialize the CF so its data register loaded with 512-byte of boot sector is mapped to 0x0-0x1FF. The actual code stored in boot sector is called CFMonLdr, which is a series of in-line codes that copy the CFMon op code into memory starting at 0x1000 and then jump into 0x1000 at the end of the copy. This is necessary because the 512-byte data in the FIFO of CF data register is destroyed with each read so no looping instructions are possible. Because CFMon executes in 0x1000, it must be 'org' at 0x1000 and the resulting op codes hand assembled into CFMonLdr. This is a obscure and tedious process best illustrated with the actual code,CFMonLdr, at the end of this article.

## ZZMon build procedure
ZZMon is monitor for TinyZ280. In UART bootstrap mode, it is loaded into memory via the serial port and then copied into CF track 0 with the ZZMon command 'C1'. With ZZMon and CFMonLdr in track 0, the CF bootstrap mode can be enabled so ZZMon will run on next power cycle or reset.

ZZMon starts at location 0x400 and can be no bigger than 3K (0x400-0xFFF), including the stack. The CFMonLdr is appended to ZZMon starting at location 0x200. Since CFMonLdr must fit in the boot sector, it is 512 byte or less (0x200-0x3FF). The ZZMon command 'C0' will copy code in 0x200-0x3FF into boot sector. The ZZMon command 'C1“ will copy code from 0x400-0xFFF to CF LBA sector 0xF8-0xFD.

## CPM2.2 build procedure
The source (Z80 version) for cpm22all is from here:
http://cpm.z80.de/download/cpm2-asm.zip
The source contains the CCP and BDOS. The custom BIOS for TinyZ280 is appended to the original source and assembled using Zilog ZDS. The resulting Intel Hex records are loaded in memory 0xDC00-0xFFFF. The ZZMon command “C2” copies data in 0xDC00-0xFFFF to CF LBA sector 0x80-0x92. Conversely, ZZMon command “B2” restores data in 0xDC00-0xFFFF from CF's LBA sector 0x80-0x92 and jumps to 0xF200 which is the starting point of CPM2.2

## CPM3 CPMLDR build procedure
Unlike the cpm22all which contains the CCP, BDOS, and BIOS, CPM3 cpmldr is a skelton of cpm3 whose only job is to find CPM3.SYS in drive A, load and execute it. LDRBIOS is created according to the CP/M Plus System Guide and helps from members of retrobrewcomputers and VCFed. LDRBIOS is assembled with zmac:

**zmac –rel ldrbios**

The resulting ldrbios.rel is transferred into CP/M2.2 environment and linked to execute at location 0x1100

**link cpmldr[L1100]=cpmldr,ldrbios**

The resulting cpmldr.com is transferred out of the CP/M2.2 environment to PC where it is converted to Intel Hex format:

**bin2hex /O0x1100 cpmldr.com cpmldr.hex**

The ZZMon Hex loader loads cpmldr.hex to 0x1100 and ZZMon command “C3” copies the data in memory to CF's LBA 0x1-0xF. Conversely ZZMon command “B3” restores data in memory 0x1100-0x2CFF from CF's LBA 0x1-0xF and jumps into 0x1100.

## LoadnGo build procedure
LoadnGo.run is loaded only when TinyZ280 is in UART bootstrap mode (mode jumper removed). This file will load ZZMon into memory and execute it after the load is completed. LoadnGo.run is concatenation of two programs:

1. A small Intel Hex file loading program that is exactly 256 bytes long. If the actual program is lesser than 256 bytes, it must be padded with any values to 256 bytes. The reason it must be 256 bytes is because the Z280 in UART bootstrap mode expects 256 bytes of binary bootstrap code to be loaded via the serial port. The code thus loaded is stored in memory location 0x0-0xFF. When the 256-th byte is received, the CPU begins program execution at location 0x0.

2. ZZMon in Intel Hex format which is appended to the small file loader described above. After Z280 has received 256 bytes of binary bootstrap code, it executes the bootstrap code while the serial port continues to receive data that is the beginning of ZZMon in Intel Hex format. The bootstrap code is an Intel Hex file loader that reads the incoming data record, verifies the checksum, and copies it to memory. When the end of Intel Hex record is detected, the file loader jumps to location 0x400 which is the starting point of ZZMon.

LoadnGo.run is concatenation of loadngo.bin and ZZMon.hex created in a DOS window with the copy command:

**copy loadngo.bin + zzmon.hex LoadnGo.run**

### Listing of CFMonLdr
```
; This is the cold bootstrap that loads the CF copying program to 0x1000 and then jump to 0x1000.
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
; CFMonLdr, Copyright (C) 2018 Hui-chien Shen
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
    org 0
    ld hl,1000h    ; create CFMon in 0x1000
    ld (hl),3eh    ; op code for LD A,40h
    inc hl
    ld (hl),40h
    inc hl
    ld (hl),0d3h   ; op code for OUT (CF2427),A
    inc hl
    ld (hl),0cdh
    inc hl
    ld (hl),16h    ; op code for LD D,0F8h
    inc hl
    ld (hl),0f8h
    inc hl
    ld (hl),21h    ; op code for LD HL,400h
    inc hl
    ld (hl),0h
    inc hl
    ld (hl),04h
    inc hl
    ld (hl),0eh    ;op code for LD C,CFdata
    inc hl
    ld (hl),0c0h
    inc hl
    ld (hl),3eh    ;op code for LD A,1
    inc hl
    ld (hl),1h
    inc hl
    ld (hl),0d3h   ;op code for OUT (CFsectcnt),A
    inc hl
    ld (hl),0c5h
    inc hl
    ld (hl),7ah    ;op code for LD A,D
    inc hl
    ld (hl),0feh   ;op code for CP 0FEh
    inc hl
    ld (hl),0feh
    inc hl
    ld (hl),0cah   ;op code for JP Z,400h
    inc hl
    ld (hl),0h
    inc hl
    ld (hl),04h
    inc hl
    ld (hl),0d3h   ;op code for OUT (CF07),A
    inc hl
    ld (hl),0c7h
    inc hl
    ld (hl),3eh    ;op code for LD A,20h
    inc hl
    ld (hl),20h
    inc hl
    ld (hl),0d3h   ;op code for OUT (CFstat),A
    inc hl
    ld (hl),0cfh
    inc hl
    ld (hl),0dbh   ;op code for IN A,(CFstat)
    inc hl
    ld (hl),0cfh
    inc hl
    ld (hl),0e6h   ;op code for AND 8
    inc hl
    ld (hl),8h
    inc hl
    ld (hl),0cah   ;op code for JP Z,101bh
    inc hl
    ld (hl),1bh
    inc hl
    ld (hl),10h
    inc hl
    ld (hl),6h     ;op code for LD B,0h
    inc hl
    ld (hl),0h
    inc hl
    ld (hl),0edh   ;op code for INIRW
    inc hl
    ld (hl),92h
    inc hl
    ld (hl),0dbh   ;op code for IN A,(CFstat)
    inc hl
    ld (hl),0cfh
    inc hl
    ld (hl),14h    ;op code for INC D
    inc hl
    ld (hl),0c3h   ;op code for JP 100bh
    inc hl
    ld (hl),0bh
    inc hl
    ld (hl),10h
    jp 1000h       ; start CFMon execution at 0x1000
    END
; This is the CFMon program that'll be created in 0x1000 by the CFMonLdr
                             A       16     org 1000h
00001000 3E 40               A       18     ld a,40h         ; LA addressing mode
00001002 D3 CD               A       19     out (CF2427),a
00001004 16 F8               A       20     ld d,0f8h        ; points to sector to read, last 8 sectors in track 0
00001006 21 00 04            A       21     ld hl,400h       ; store CF data starting from 400h
00001009 0E C0               A       22     ld c,CFdata      ; reg C points to CF data reg
0000100B                     A       23 moresect:
0000100B 3E 01               A       24     ld a,1           ; read 1 sector
0000100D D3 C5               A       25     out (CFsectcnt),a    ; write to sector count with 1
0000100F 7A                  A       26     ld a,d           ; read sector pointed by reg D
00001010 FE FE               A       27     cp 0feh          ; read sectors 0xF8-0xFD
00001012 CA 00 04            A       28     jp z,400h        ; load completed, run program loaded at 0x400
00001015 D3 C7               A       29     out (CF07),a     ; read the sector pointed by reg D
00001017 3E 20               A       34     ld a,20h         ; read sector command
00001019 D3 CF               A       35     out (CFstat),a   ; issue the read sector command
0000101B                     A       36 readdrq:
0000101B DB CF               A       37     in a,(CFstat)    ; check data request bit set before read CF data
0000101D E6 08               A       38     and 8            ; bit 3 is DRQ, wait for it to set
0000101F CA 1B 10            A       39     jp z,readdrq
                             A       40
00001022 06 00               A       41     ld b,0h          ; sector has 256 16-bit data
00001024 ED 92               A       42     db 0edh,92h      ; op code for inirw input word and increment
                             A       43 ;    inirw           ; reg HL and c are already setup at the top
00001026 DB CF               A       44     in a,(CFstat)    ; OUTJMP bug fix
00001028 14                  A       45     inc d
00001029 C3 0B 10            A       46     jp moresect
```

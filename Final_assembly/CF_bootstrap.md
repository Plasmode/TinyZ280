# Bootstrap from Compact Flash
This is the design description of cold boot software out of a compact flash.

The motivation is two folds. First is cost saving since a set of ROM is not required resulting in smaller pc board and fewer parts. Second is ease of programming that CF can easily reprogrammed in-situ.

### Introduction
This concept works the best for processor with 16-bit wide data bus because it matches with CF's native 16-bit data bus. At power up (or when reset button is pressed), a state machine (CFinit) holds the CPU in reset while configure the CF to read the boot sector (first sector of a CF disk). The boot sector contains a cold bootstrap code that copies a CF loader into memory and jump to it. After CF is configured to read the boot sector, the state machine also configure the memory map so memory location 0-0x1FF are mapped to the CF's data register where content of the boot sector will stream out with each CPU read. The state machine then negates CPU RESET and CPU begin fetch instruction starting from location 0. Because the CF data register is basically a 256 words deep FIFO that the cold bootstrap code must be written such that there are no looping. The state machine contains a register that can be cleared by software which will restore the memory map of 0-0x1FF to normal memory.

### CFinit State Machine
State Transition:

1. read CF Status register bit 7 (BSY) & bit 6 (DRDY) until (BSY==0 AND DRDY==1)
2. Write 0x1 to CF Sector Count register
3. Write 0x1 to CF Sector register
4. Write 0x0 to CF Cylinder Low register
5. Write 0x0 to CF Cylinder High register
6. Write 0x0 to CF Drive/Head register
7. Write 0x20 (Read) to CF Command register
8. Read CF Status register bit 7 (BSY) & bit 3 (DRQ) until (BSY==0 AND DRQ==1)
9. Remap 0x0 - 0x1FF to CF Data register and Release RESET to Z280
### Memory Map Configuration
Cold Bootstrap Code
This is the code that will be in memory once cold bootstrap is done loading:
```
; 2/10/2018
; copy program in track 0, sector 2 & 3 and run
CFdata       equ 0C0h        ;CF data register
CFerr        equ 0C2h        ;CF error reg
CFsectcnt    equ 0C5h        ;CF sector count reg
CF07         equ 0C7h        ;CF LA0-7
CF815        equ 0C9h           ;CF LA8-15
CF1623       equ 0CBh           ;CF LA16-23
CF2427       equ 0CDh           ;CF LA24-27
CFstat       equ 0CFh           ;CF status/command reg
    org 1000h
;    ld sp,0FFFFh    ; initialize stack
    ld d,2        ; points to sector to read
    ld hl,800h    ; store CF data starting from 800h
    ld c,CFdata   ; reg C points to CF data reg
moresect:
    ld a,1        ; read 1 sector
    out (CFsectcnt),a    ; write to sector count with 1
    ld a,d        ; read sector pointed by reg D
    cp 4          ; read sector 2 & 3 only
    jp z,800h        ; load completed, run program
    out (CF07),a     ; read the sector pointed by reg D
; no need to setup track, it is setup by CFinit state machine
;    xor a        ; clear reg A
;    out (CF1623),a    ; track 0
;    out (CF815),a
    ld a,20h          ; read sector command
    out (CFstat),a    ; issue the read sector command
readdrq:
    in a,(CFstat)    ; check data request bit set before read CF data
    and 8            ; bit 3 is DRQ, wait for it to set
    jp z,readdrq

    ld b,0h        ; sector has 256 16-bit data
    db 0edh,92h    ; op code for inirw input word and increment
;    inirw        ; reg HL and c are already setup at the top
    in a,(CFstat)    ; OUTJMP bug fix
    inc d
    jp moresect
```
This is the cold bootstrap code that executes the FIFO instruction stream from CF and creates the code above, byte-by-byte. It is strictly an in-line code with no looping and ends with a jump to the code just created.


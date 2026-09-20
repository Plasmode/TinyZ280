# TinyZ280, Prototyping the Z280 in 100mm X 100mm PC Board (AKA TinyZZ)
## Introduction
I'm new to Z80. I looked at Z280 more closely after a round of interesting discussion with Lamar Owen and really like what I read. The prototype TinyZ280 board is the result of that discussion. It is a pathfinder board; it has multiple ways of booting up and different RAM choices so I can learn and hopefully get smart along the way. Another goal of this prototype is a stepping stone to TinyZZ, a low cost disposable Z280-based computer to experiment with. I don't want to tear down experiments just to salvage and re-use the computer. To keep the cost down and setting up experiment easier, it (TinyZZ) will not have boot ROM or flash. It will boot out of CF. The UART bootstrap feature is used to boot up the first time and configure the CF. CPLD is large enough to have spare pins and logics to support experiments. This prototype board is the vehicle to investigate how feasible the TinyZZ concepts are.

![topview](TinyZ280_topview.jpg)
## Design Concept
Schematic of TinyZ280. There are 4 different ways to boot up the Z280:

1. UART bootstrap. This method is based on Z280's UART bootstrap feature where nWAIT signal is asserted and the value 0x40 is presented on the data bus AD[7:0] when nRESET signal is negated. This will cause Z280 to enter its UART bootstrap mode where it will configure DMA channel 0 to transfer 256 bytes of data from UART to memory location 0-0xFF. Once 256 bytes of data are received, it will release the reset to Z280 CPU and program execution will start at location 0. The serial baud rate is determined by the clock to counter channel 1 divided by 16. The parity is set to odd. The current serial configuration is 57600, odd parity, 8 data bits, 1 stop. This configuration requires either SIMM DRAM or two 128Kx8 RAM in U3 and U4
2. Flash memory. This is the conventional method of booting up. Jumper block J3-J6 are configured as J3-J4, J5-J6, and two 29F010 flash memory devices are programed with the appropriate software and inserted in U3 and U4. This configuration requires SIMM DRAM populated as the volatile memory.
3. CF flash. The bootstrap code is in track 0, sector 1 of the compact flash drive. After reset, memory locations 0-0xFF are mapped to the 16-bit wide data register of the compact flash drive. After streaming 256 bytes of instructions from the CF, Z280 will have copied enough code to execute a small bootstrap that will bring in rest of the software from CF. This configuration requires either static RAM in U3/U4 or DRAM in the SIMM socket.
4. Serial EEPROM. The serial EEPROM, AT24C256, contains 32K bytes of monitor/debugger software. At power up, the CPLD holds Z280 in reset and copy the content of the serial EEPROM into RAM or DRAM starting from memory location 0. When the copy is completed, the Z280 reset is released and execution starts from location 0. This configuration requires RAM in U3/U4 or DRAM in the SIMM socket.

There are two RAM configurations:
1. Static RAM. U3 and U4 hosts two 128K X 8 static RAM. The jumper block J3-J6 are set as follow: J3-to-J6, J4-to-J5
2. Dynamic RAM. Populate the SIMM socket with up to 16 megabyte of 72-pin SIMM DRAM.
There are still pc board space available after the above requirements are fulfilled. So it is filled with a serial device, COM81C17 and a real-time clock, DS12887.

The key to the design is Altera's EPM7128 CPLD. Different configurations will require a different CPLD design. The first configuration is to boot up using UART bootstrap. Once UART bootstrap is successful and the appropriate software are developed, it will program the CF's track 0 sector 1 with the CF bootstrap code. Then Altera CPLD is reprogrammed with new design to bootstrap out of CF's track 0/sector 1. The volatile memory will be SRAM during the development period because it is easier to debug with static RAM. At the end of the development period the SRAM will be replaced with cheaper DRAM.

## Implementation
![base](TinyZ280_unpopulated_baseboard.jpg)
PC board design files are here. The pc board is 100mm x 100mm, 1.2mm thick. It was manufactured by Seeed Studio.

Construction notes is here.

### Step 1, UART Bootstrap

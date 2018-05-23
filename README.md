# Notebook
Things to remember while writing an OS

## Steps
- Install a machine emulator like `qemu`.
- Boot sector
  - Write boot sector code using a binar editor, or a simple assembler code.
  - Compile
  - Run using qemu.
- Make boot sector print some text by raising an interrupt for it.

## Concepts
#### qemu
A generic and open source machine emulator and virtualizer.

#### Assembler
A computer program which translates assembly language to an object file or machine language format.

#### BIOS
- The program a personal computer's microprocessor uses to get the computer system started after you turn it on.
- Also manages data flow between the computer's operating system and attached devices.

#### Boot sector
- Region of a data storage device that contains machine code to be loaded into random-access memory (RAM) by a computer system's built-in firmware.
- When the computer boots, the BIOS doesn't know how to load the OS, so it delegates that task to the boot sector.
- Thus, the boot sector must be placed in a known, standard location.
- That location is the first sector of the disk (cylinder 0, head 0, sector 0) and it takes 512 bytes.
- To make sure that the "disk is bootable", the BIOS checks that bytes 511 and 512 of the alleged boot sector are bytes 0xAA55.
- This is the simplest boot sector ever:

```
e9 fd ff 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
[ 29 more lines with sixteen zero-bytes each ]
00 00 00 00 00 00 00 00 00 00 00 00 00 00 55 aa
```
It is basically all zeros, ending with the 16-bit value 0xAA55 (beware of endianness, x86 is little-endian). The first three bytes perform an infinite jump
- Can either write the above 512 bytes with a binary editor, or just write a very simple assembler code

```assembly
; Infinite loop (e9 fd ff)
loop:
    jmp loop

; Fill with 510 zeros minus the size of the previous code
times 510-($-$$) db 0
; Magic number
dw 0xaa55
```
  - To compile: `nasm -f bin boot_sect_simple.asm -o boot_sect_simple.bin`
  - To run: `qemu boot_sect_simple.bin`

#### Big-endian and Little-endian
- These terms describe the order in which a sequence of bytes are stored in computer memory. 
- Big-endian is an order in which the "big end" (most significant value in the sequence) is stored first (at the lowest storage address). 
- Little-endian is an order in which the "little end" (least significant value in the sequence) is stored first. 
- For example:
  - in a big-endian computer, the two bytes required for the hexadecimal number 4F52 would be stored as 4F52 in storage (if 4F is stored at storage address 1000, for example, 52 will be at address 1001). 
  - In a little-endian system, it would be stored as 524F (52 at address 1000, 4F at 1001).

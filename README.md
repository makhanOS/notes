# Notebook
Things to remember while writing an OS

## Table of Contents

- [Notebook](#notebook)
  - [Table of Contents](#table-of-contents)
  - [Steps](#steps)
  - [Q&A](#qa)
      - [Why hexadecimal is used in low-level programming](#why-hexadecimal-is-used-in-low-level-programming)
      - [Oldest CPU implementation?](#oldest-cpu-implementation)
  - [Concepts](#concepts)
      - [qemu](#qemu)
      - [Assembler](#assembler)
      - [BIOS](#bios)
      - [Boot sector](#boot-sector)
      - [Big-endian and Little-endian](#big-endian-and-little-endian)
      - [Interrupts](#interrupts)
      - [CPU registers](#cpu-registers)
      - [Memory offsets](#memory-offsets)
      - [Pointers](#pointers)
      - [Interrupts](#interrupts)
      - [Stack](#stack)
      - [Strings](#strings)
      - [Control structures](#control-structures)
      - [Calling functions](#calling-functions)
      - [Including external files](#including-external-files)
      - [Segmentation](#segmentation)

## Steps

- Install a machine emulator like `qemu`.
- Boot sector
  - Write boot sector code using a binar editor, or a simple assembler code.
  - Compile
  - Run using qemu.
- Make boot sector print some text by raising an interrupt for it.
- Declaring some data in our program and using it like a variable.
  - while doing so, we prefix it with a label
  - can put labels anywhere in our program
  - 

## Q&A

#### Why hexadecimal is used in low-level programming

Since it is a power of 4, BIOS data can be easily interpreted by considering every digit.

#### Oldest CPU implementation?
Intel 8086:  had support for 16-bit instructions and no notion of memory protection.

## Concepts

#### qemu

- A generic and open source machine emulator and virtualizer.
- Better that some other emulators because it can run architectures other than the `x86` too.

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

  - It is basically all zeros, ending with the 16-bit value 0xAA55 (beware of endianness, x86 is little-endian). The first three bytes perform an infinite jump
  - About this boot sector:
    - The initial three bytes, in hexadecimal as 0xe9, 0xfd and 0xff, are actually machine code instructions, as defined by the CPU manufacturer, to perform an endless jump.
    - The last 2 bytes `0x55` and `0xaa` make up the magic number, that tells the BIOS that this is a boot block and not just some data that happens to be in the boot sector.
    - The file is padded with zeros in the middle just so that the magic number appears at the end.
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
- In our OS we are using little-endianness because `x84` architecture uses that by default

#### Interrupts

An interrupt is a signal from a device attached to a computer or from a program within the computer that requires the operating system to stop and figure out what to do next.

#### CPU registers

- A processor register is a quickly accessible location available to a computer's central processing unit (CPU). 
- Registers usually consist of a small amount of fast storage, although some registers have specific hardware functions, and may be read-only or write-only.
- we can store data temporarily during a particular routine
- All x86 CPUs have four general purpose registers: ax, bx, cx, and dx
- Also, these registers, which can each hold a word (two bytes, 16 bits) of data, can be read and written by the CPU with negligible delay as compared with accessing main memory
- In assembly programs, one of the most common operations is moving (or more accurately, copying) data between these registers:

```assembly
mov ax , 1234 ; store the decimal number 1234 in ax
mov cx , 0 x234 ; store the hex number 0 x234 in cx
mov dx , ’t ’ ; store the ASCII code for letter ’t’ in dx
mov bx , ax ; copy the value of ax into bx , so now bx == 1234
```

- Notice that the destination is the first and not second argument of the mov operation, but this convention varies with different assemblers.
- Sometimes it is more convenient to work with single bytes, so these registers let us set their high and low bytes independently:

```assembly
mov ax , 0 ; ax -> 0x0000 , or in binary 0000000000000000
mov ah , 0 x56 ; ax -> 0 x5600
mov al , 0 x23 ; ax -> 0 x5623
mov ah , 0 x16 ; ax -> 0 x1623
```

- Four 32-bit data registers are used for arithmetic, logical, and other operations. These 32-bit registers can be used in three ways −
  - As complete 32-bit data registers: EAX, EBX, ECX, EDX.
  - Lower halves of the 32-bit registers can be used as four 16-bit data registers: AX, BX, CX and DX.
  - Lower and higher halves of the above-mentioned four 16-bit registers can be used as eight 8-bit data registers: AH, AL, BH, BL, CH, CL, DH, and DL.
- Some of these data registers have specific use in arithmetical operations.
  - **AX is the primary accumulator**; it is used in input/output and most arithmetic instructions. For example, in multiplication operation, one operand is stored in EAX or AX or AL register according to the size of the operand.
  - **BX is known as the base register**, as it could be used in indexed addressing.
  - **CX is known as the count register**, as the ECX, CX registers store the loop count in iterns.ative operatio
  - **DX is known as the data register**. It is also used in input/output operations. It is also used with AX register along with DX for multiply and divide operations involving large values.

#### Memory offsets

- In computer science, an offset within an array or other data structure object is an integer indicating the distance (displacement) between the beginning of the object and a given element or point, presumably within the same object.
- The concept of a distance is valid only if all elements of the object are of the same size (typically given in bytes or words).
- For example, in A as an array of characters containing "abcdef", the fourth element containing the character 'd' has an offset of three from the start of A.

#### Pointers

a pointer is a programming language object, whose value refers to (or "points to") another value stored elsewhere in the computer memory using its memory address.

#### Interrupts

- mechanism that allows CPU to temporarily halt what it is doing and then run some other high priority task before returninig to the original task.
- can be raised by either a software instruction (eg. `int 0x10`) or by some hardware device.
- each interrupt represented by a unique number that is an index to the interrupt vector, a table initially set up by BIOS at the start of memory that contains address pointers to interrupt service routines (ISRs)
- An ISR is simply a sequence of machine instructions, much like our boot sector code, that deals with a specific interrupt
- BIOS adds some of its own ISRs to the interrupt vector that specialise in certain aspects of the computer, for example: interrupt 0x10 causes the screen-related ISR to be invoked; and interrupt 0x13, the disk-related I/O ISR.
- However, it would be wasteful to allocate an interrupt per BIOS routine, so BIOS multiplexes the ISRs by what we could imagine as a big switch statement, based usually on the value set in one of the CPUs general purpose registers, ax, prior to raising the interrupt.

#### Stack

- as in simple stack
- can be initialized by defining a stack pointer, like: `mov sp, bp`
- here `bp` is the address of the stack from where we want to start storing the values
- can simply push into stack using: `push 'C'`
- while accessing, start from behind, not from where the memory started.
- Can print stack values one by one like this:

```assembly
pop bx
mov al, bl
int 0x10 ; prints C
```

- The stack is implemented by two special CPU registers, bp and sp, which maintain the addresses of the stack base (i.e. the stack bottom) and the stack top respectively. 
- Since the stack expands as we push data onto it, we usually set the stack’s base far away from important regions of memory (e.g. such as BIOS code or our code) so their is no danger of overwriting if the stack grows too large.
- One confusing thing about the stack is that it actually grows downwards from the base pointer, so when we issue a push, the value actually gets stored below --- and not above --- the address of bp, and sp is decremented by the value’s size.

#### Strings

- can define strings like we defined chars and ints
- but should specify the end, like we do in C
- sample string initialization:

```assembly
mystring:
    db 'Hello, World', 0
```

- Text surrounded with quotes is converted to ASCII by the assembler, while that lone zero will be passed as byte 0x00 (null byte)

#### Control structures

- We have already used one: `jmp $` for the infinite loop.
- Assembler jumps are defined by the previous instruction result. For example:

```assembly
cmp ax, 4      ; if ax = 4
je ax_is_four  ; do something (by jumping to that label)
jmp else       ; else, do another thing
jmp endif      ; finally, resume the normal flow

ax_is_four:
    .....
    jmp endif

else:
    .....
    jmp endif  ; not actually necessary but printed here for completeness

endif:
```

- Think in your head in high level, then convert it to assembler in this fashion.
- There are many jmp conditions: if equal, if less than, etc. They are pretty intuitive but can always Google them

#### Calling functions

- calling a function is just a jump to a label
- The tricky part are the parameters. There are two steps to working with parameters:
  - The programmer knows they share a specific register or memory address
  - Write a bit more code and make function calls generic and without side effects
- Step 1 is easy. Let's just agree that we will use al (actually, ax) for the parameters.

```assembly
mov al, 'X'
jmp print
endprint:

...

print:
    mov ah, 0x0e  ; tty code
    int 0x10      ; I assume that 'al' already has the character
    jmp endprint  ; this label is also pre-agreed
```

- You can see that this approach will quickly grow into spaghetti code. The current print function will only return to endprint. What if some other function wants to call it? We are killing code reusage.
- The correct solution offers two improvements:
  - We will store the return address so that it may vary
  - We will save the current registers to allow subfunctions to modify them without any side effects
- To store the return address, the CPU will help us. Instead of using a couple of `jmp` to call subroutines, use `call` and `ret`
- To save the register data, there is also a special command which uses the stack: `pusha` and its brother `popa`, which pushes all registers to the stack automatically and recovers them afterwards.

#### Including external files

```assembly
%include "file.asm"
```

#### Segmentation

- Means that you can specify an offset to all the data you refer to
- Done by using special registers: `cs`, `ds`, `ss`, `es` for Code, data, stack, extra (user-defined)
- to compute the real address we don't just join the two addresses, but we overlap them: `segment << 4 + address`.
  - For example, if `ds` is `0x4d`, then `[0x20]` actually refers to `0x4d0 + 0x20 = 0x4f0`

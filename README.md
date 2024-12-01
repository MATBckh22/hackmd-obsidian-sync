# Assembly Notes

[Reference](https://www.tutorialspoint.com/assembly_programming/index.htm)

## The Assembly Language

The Assembly Language (ASM) is a low-level language that involves writing instructions for the processor to execute directly. It is the lowest level apart from machine code to communicate with the machine since these instructions represent symbolic code of machine language.

We will be using x86 ASM in DosBox through MASM. 

## DosBox and MASM

DosBox is an emulator that supports the x86 assembly language. 

After installation, run this command to mount it to ur directory:

```
mount c d:\masm
c:
```

U may check if DosBox succesfully detected ur MASM by typing:
```
MASM
```

Ur output should look like this:

![image](https://hackmd.io/_uploads/H1l-T7ym1x.png)

Additionally, u can put this at the bottom of the options script to skip this process whenever u open DosBox.

## Program Structure

Assembly programs are composed of three sections:
- `DATA SEGMENT`
- `EXTRA SEGMENT`
- `CODE SEGMENT`

### `DATA` Segment

This section is used for declaring initialized data or constants. It doesn't change at runtime, so u can declare constant values, file names, etc.

We declare the data section as:

```masm
DATA SEGMENT
[label] DB "Hello World"
DATA ENDS
```

Note that:
- `DW`: Define Word
    - integer
- `DB`: Define Byte
    - strings and arrays
- `DD`: Double Word
    - large integers

### `EXTRA` Segment

This section is used for declaring variables:

```masm
EXTRA SEGMENT
[label] DW 25
EXTRA ENDS
```

### `CODE` Segment

This is where we write the code.

```masm
CODE SEGMENT
ASSUME CS:CODE, DS:DATA, ES:EXTRA
START: MOV AX, DATA ; initialization
       MOV DS, AX
       ; needed if there's an EXTRA segment
       MOV AX, EXTRA
       MOV ES, AX
       ...
CODE ENDS
     END START
```

## Statements/Instructions

ASM programs operate on three types of statements:

- instructions
    - instructions that directly tell the procesor what to do
    - each instruction consists of an operation code (opcode)
- assembler directives or pseudo-ops
    - various aspects of the assembly process
    - non-executable
- macros
    - textual substitution

## Syntax

ASM statements follows this format:

```
[label] mnemonic [operands] [;comment]
```

Fields with square brackets are optional. An instruction consists of the name of the instruction (mnemonic) or **to be executed** followed by operands.

```nasm
INC COUNT ;increment memory variable count
MOV TOTAL, 48 ;transfer the value 48 in memory variable TOTAL
ADD AH, BH ;add content of BH into AH register
AND MASK1, 128 ;perform AND operation on variable MASK1 and 128
ADD MARKS, 10 ;add 100 to variable MARKS
MOV AL, 10 ;transfer the value 10 into AL register
```

### Printing Hello World

We use this example as the simplest way to introduce how an ASM program works:

```nasm
DATA SEGMENT
MSG DB "Hello World", "$" ;$ indicates an End of String argument
DATA ENDS

CODE SEGMENT
ASSUME CS:CODE, DS:DATA ; program initialization
START: MOV AX, DATA
	   MOV DS, AX
	   
	   MOV DX, OFFSET MSG ; loads offset msg to dx
	   ; after loading MSG's offset into DX, full address of MSG can be
	   ; referenced as DS:DX
	   MOV AH, 09H ; DOS function to print string
	   INT 21H ; DOS interruption to execute instruction
	   
	   MOV AH, 4CH ; sets up DOS function for program termination
	   INT 21H ; terminates program (from MOV AH, 4CH)
	   
CODE ENDS
	 END START
```

Save this file as `[filename].asm` and put it into the directory u have mounted MASM with, exp `D:\MASM`. Then, type:

```
masm [filename].asm
```

This just assembles the ASM file, we have to run it using:

```
link [filename].obj
[filename]
```

The output should look like this:

![image](https://hackmd.io/_uploads/H1CJLIemyx.png)

## General Purpose Registers
| **Register** | **Full Name**        | **Purpose**                                        | **Common Instructions**                                           |
|--------------|----------------------|---------------------------------------------------|--------------------------------------------------------------------|
| **AX**       | Accumulator Register | Used for arithmetic operations, I/O instructions | `MOV AX, value` / `ADD AX, BX` / `MUL CX` / `INT 21H`             |
| **BX**       | Base Register        | Used for addressing memory                        | `MOV BX, OFFSET DATA` / `ADD BX, AX` / `MOV [BX], AL`             |
| **CX**       | Count Register       | Loop control and shift count                      | `MOV CX, count` / `LOOP label` / `SHL AX, CL`                     |
| **DX**       | Data Register        | Used for I/O operations, extended multiplication/division | `MOV DX, value` / `DIV BX` / `OUT DX, AL`                     |

---

## Segment Registers
| **Register** | **Full Name**           | **Purpose**                             | **Common Instructions**                                           |
|--------------|-------------------------|-----------------------------------------|--------------------------------------------------------------------|
| **CS**       | Code Segment            | Points to the code being executed       | `ASSUME CS:CODE` / `MOV AX, CS`                                   |
| **DS**       | Data Segment            | Points to the data segment              | `ASSUME DS:DATA` / `MOV DS, AX`                                   |
| **SS**       | Stack Segment           | Points to the stack segment             | `ASSUME SS:STACK` / `MOV SS, AX`                                  |
| **ES**       | Extra Segment           | Extra pointer for string/data operations | `MOV ES, AX` / `MOV [ES:DI], AL`                                  |

---

## Index and Pointer Registers
| **Register** | **Full Name**       | **Purpose**                              | **Common Instructions**                                           |
|--------------|---------------------|------------------------------------------|--------------------------------------------------------------------|
| **SI**       | Source Index        | Source for string operations             | `MOV AL, [SI]` / `LODSB` / `ADD SI, 1`                            |
| **DI**       | Destination Index   | Destination for string operations        | `MOV [DI], AL` / `STOSB` / `ADD DI, 1`                            |
| **BP**       | Base Pointer        | Points to the base of the stack frame    | `MOV BP, SP` / `MOV AX, [BP+4]`                                   |
| **SP**       | Stack Pointer       | Points to the top of the stack           | `PUSH AX` / `POP BX` / `MOV AX, [SP]`                             |
| **IP**       | Instruction Pointer | Points to the next instruction           | Automatically updated during execution                            |

---

## Special Purpose Registers
| **Register** | **Purpose**                                        | **Common Instructions**                                           |
|--------------|----------------------------------------------------|--------------------------------------------------------------------|
| **FLAGS**    | Holds status flags for comparisons, arithmetic, etc. | `CMP AX, BX` / `JZ label` / `JC label`                            |

---

## Common Instructions

### Data Transfer
- `MOV dest, src` — Transfers data from `src` to `dest`.
- `PUSH reg/mem` — Pushes a register/memory value onto the stack.
- `POP reg/mem` — Pops the top value of the stack into a register/memory.

### Arithmetic
- `ADD reg/mem, value` — Adds a value to a register/memory.
- `SUB reg/mem, value` — Subtracts a value from a register/memory.
- `MUL reg` — Multiplies `AX` by a register (result in `AX` or `DX:AX` for larger values).
- `DIV reg` — Divides `AX` (or `DX:AX` for larger values) by a register.

### Logic
- `AND reg, value` — Performs a bitwise AND operation.
- `OR reg, value` — Performs a bitwise OR operation.
- `XOR reg, value` — Performs a bitwise XOR operation.
- `NOT reg` — Inverts all bits in a register.

### Control Flow
- `JMP label` — Jumps unconditionally to `label`.
- `JE/JZ label` — Jumps if equal (or zero flag is set).
- `JNE/JNZ label` — Jumps if not equal (or zero flag is not set).
- `LOOP label` — Decrements `CX` and jumps to `label` if `CX` ≠ 0.

### Stack Operations
- `PUSH reg/mem` — Pushes a value onto the stack.
- `POP reg/mem` — Pops the top value from the stack into a register/memory.

### String Operations
- `MOVSB/MOVSW` — Moves string data from `[SI]` to `[DI]` (increments/decrements both).
- `LODSB` — Loads a byte from `[SI]` into `AL`.
- `STOSB` — Stores a byte from `AL` into `[DI]`.

### Input/Output
- `IN AL, port` — Reads a byte from a hardware port into `AL`.
- `OUT port, AL` — Writes a byte from `AL` to a hardware port.

---

## Quick Tips
- **Combine Registers for Extended Precision**:
  - `AX` and `DX` are often used together for extended multiplication/division.
- **Segment Override Prefix**:
  - Use instructions like `MOV AX, [ES:DI]` to specify which segment register to use.
- **Flags Usage**:
  - After arithmetic or comparison (`CMP`), jump instructions (`JE`, `JL`, etc.) depend on the `FLAGS` register. 

Type `DEBUG` to go into assembler tool:

```
R: view registers
D and E: Display and edit memory
T: Execute instruction
U: unassemble code at a specific memory address
```

## Often Used DOS Functions (ASM)

DOS functions mainly handles I/O operations like displaying and reading data. This [reference](https://www.philadelphia.edu.jo/academics/qhamarsheh/uploads/Lecture%2021%20MS-DOS%20Function%20Calls%20_INT%2021h_.pdf) provides some great examples on functions we often use. We're starting on the basic syntax for a DOS function which follows:

```
MOV AH, (function number) ; DOS function
INT 21H ; DOS interruption (execution)
```

### Program Termination

```masm
MOV AH, 4CH
INT 21H
```

terminates the program.

### Various Outputs

- **Output String:** `09H`
- **Output Character:** `02H`, `06H` (ASCII)

Writing a letter A to output:

```masm
MOV AH, 02H
MOV DL, 'A'
INT 21H
```

Printing a character **requires loading the character to the DL register.**

Writing a string to output:

```masm
DATA SEGMENT
STR DB "Hello World", "$"
DATA ENDS
...
MOV AH, 09H
MOV DX, OFFSET STR
; we can also perform LEA DX, MSG
INT 21H
```

Printing a string **requires loading the offset address of the string to the DX register.** Alternatively, we can also use the instruction `LEA DX, MSG`.

## Conditionals

There are some important concepts we need to understand before writing conditionals in ASM.

The first concept is the **usage of Flags.** Conditional statements rely heavily on flag registers to determine comparison outputs:

| Flag Name         | Abbreviation | Description                                                                                  |
|-------------------|--------------|----------------------------------------------------------------------------------------------|
| Carry Flag        | `CF `          | Indicates a carry out from the most significant bit in an addition or a borrow in subtraction.|
| Parity Flag       | `PF`           | Set if the number of 1 bits in the least significant byte is even.                           |
| Auxiliary Carry   | `AF`           | Set when there is a carry or borrow between bit 3 and bit 4 in an arithmetic operation.      |
| Zero Flag         | `ZF`           | Set if the result of an operation is zero.                                                   |
| Sign Flag         | `SF`           | Set if the result of an operation is negative (based on the most significant bit).           |
| Overflow Flag     | `OF`           | Set if there is a signed overflow (e.g., adding two positive numbers yields a negative result).|
| Direction Flag    | `DF`           | Controls string operations; if set, decrements pointers, otherwise increments pointers.     |
| Interrupt Flag    | `IF`           | Controls whether interrupts are enabled (`IF = 1`) or disabled (`IF = 0`).                  |

**ASM conditionals are usually composed of two sections, the comparison instruction and the comparison jump.** We use `CMP` often before a jump, this sets the appropriate flags without performing operations on the data. A subtle hint that we will understand later is that all loops in ASM are executed in a `do while` loop fashion.

We will explain the variants and usages of comparison jumps in the next chapter.

### Comparison Jumps

Comparison jumps follow the general syntax of:

```
(jump instruction) label
```

**Check if Zero / Equal**

- `JE`: jump if equal
    - jumps to label if comparisons are equal

```masm
CMP AX, BX
JE label ; if AX == BX
```

- `JZ`: jump if zero
    - jumps to label if `ZF == 0` which means that the Zero Flag is set to 1
    - **compares the previous instruction whether it returns 0**

```masm
CMP AX, 5
SUB AX, 5
JZ label ; if AX == 0
```

We can see an implementation of both which follows:

```masm
DATA SEGMENT
MSG1 DB "Zero detected", 13, 10, "$" ; to print the msg and a new line
; 13 for carriage return
; 10 for line feed
MSG2 DB "A and B are equal", "$"
DATA ENDS

CODE SEGMENT
ASSUME CS:CODE, DS:DATA
START: MOV AX, DATA
	   MOV DS, AX
	   
	   MOV AX, 5
	   SUB AX, 5
	   JZ PRINT
	   JE EQUAL
	   
PRINT: MOV DX, OFFSET MSG1
	   MOV AH, 09H
	   INT 21H
	   
EQUAL: MOV DX, OFFSET MSG2
	   MOV AH, 09H
	   INT 21H
	   
	   MOV AH, 4CH
	   INT 21H
	   
CODE ENDS
	 END START
```

**Check if Not Zero / Equal**

This operation checks for the complement of zero and equality.

- `JNE`: Jump if not equal

```masm
CMP AX, BX
JNE label ; if AX != BX
```

- `JNZ`: Jump if not zero
    - checks if `ZF != 0` or Zero Flag is not set

```masm
MOV AX, 6
SUB AX, 5
JNZ label ; if AX != 0
```

We can extend the implementation by including `JNE` and `JNZ` comparison jumps:

```masm
DATA SEGMENT
MSG1 DB "Zero detected", 13, 10, "$"
MSG2 DB "A and B are equal", 13, 10, "$"
MSG3 DB "Operation doesn't return a zero", 13, 10, "$"
MSG4 DB "A and B are not equal", 13, 10, "$"
DATA ENDS

CODE SEGMENT
ASSUME CS:CODE, DS:DATA
START: MOV AX, DATA
	   MOV DS, AX
	   
	   MOV AX, 5
	   SUB AX, 5
	   JZ ZERO
	   MOV BX, 0
	   CMP AX, BX
	   JE EQUAL
	   MOV AX, 6
	   SUB AX, 5
	   JNE NOT_ZERO
	   MOV BX, 0
	   CMP AX, BX
	   JNZ NOT_EQUAL
	   
ZERO: MOV DX, OFFSET MSG1
	   MOV AH, 09H
	   INT 21H
	   
EQUAL: MOV DX, OFFSET MSG2
	   MOV AH, 09H
	   INT 21H
	   
NOT_ZERO: MOV DX, OFFSET MSG3
		  MOV AH, 09H
		  INT 21H

NOT_EQUAL: MOV DX, OFFSET MSG4
		   MOV AH, 09H
		   INT 21H
		   
		   MOV AH, 4CH
		   INT 21H
CODE ENDS
	 END START
```

**Jump if Greater**

- `JG`: Jump **if first operand is greater than second operand**

```masm
CMP AX, BX
JG label ; if AX > BX
```

Alternatively, we can use `JNLE` which stands for jump if not less or equal. These jumps depend on Sign Flag (SF) and Zero Flag (ZF).

**Jump if Greater or Equal**

- `JGE`: Jump if first operand is greater than or equal second operand

```masm
CMP AX, BX
JGE label ; if AX >= BX
```

**Jump if Lesser**

- `JL`: Jump if first operand is lesser than second operand

```masm
CMP AX, BX
JL label ; if AX < BX
```

**Jump if Lesser or Equal**

- `JLE`: Jump if first operand is lesser than or equal to second operand

```masm
CMP AX, BX
JLE label ; if AX <= BX
```

**Jump if Carry / Not Carry**

`JC` and `JNC` are used to determine unsigned overflow:

- `JC`: jump if the operations results in a carry
- `JNC`: jump if no carry occurs

```masm
ADD AX, BX
JC label ; jump if adding BX to AX results in a carry (unsigned overflow)
```

## Loops

Loops in ASM operate in a `do-while` fashion, having to execute a block of code before evaluating the condition. We can appreciate C's abstraction of for loops since they take more effort to write in ASM.

 ### For Loops

Recall that in C we define for loops in this manner:

```c
for (int i = 0; i < 10; i++){
    //do something
}
```

In ASM, we use Count Register (CX) to keep track of the loop. This example of printing the character `*` shows us how to implement it:

```masm
DATA SEGMENT
CHAR DB "*", "$"
DATA ENDS

CODE SEGMENT
ASSUME CS:CODE DS:DATA
START: MOV AX, DATA
	   MOV DS, AX
	   
	   MOV CX, 10 ; set loop counter to 10
	   
PRINT: MOV AH, 02H ; invoke DOS function to print character
	   MOV DL, CHAR ; load CHAR
	   INT 21H
	   LOOP PRINT ; execute loop by jumping to PRINT until CX == 0
	   
	   MOV AH, 4CH ; program termination
	   INT 21H
CODE ENDS
	 END START   
```

Alternatively, we can replace `LOOP PRINT` by decrementing CX using `DEC CX` and jumping while `CX != 0` using `JNZ LOOP`.

### While Loops

We provide a simple declaration of while loops in C:

```c
while (x == 1){
    //do something
}
```

Since while loops need a condition to evaluate to determine whether or not to stop the program, we can use our previously explained Comparison Jumps to define conditions. **An extra label is needed to evaluate the condition:**

```masm
DATA SEGMENT
NUM1 DW 5
NUM2 DW 8
STRING1 DB "Decrementing", 13, 10, "$"
STRING2 DB "Now NUM1 == NUM2", "$"
DATA ENDS

CODE SEGMENT
ASSUME CS:CODE DS:DATA
START: MOV AX, DATA
	   MOV DS, AX
	   
	   MOV AX, NUM1
	   MOV BX, NUM2

CONDITION: CMP AX, BX
		   JE END_LOOP

LOOP1:  DEC BX
	    MOV AH, 09H
	    MOV DX, OFFSET STRING1
	    INT 21H
	    JMP CONDITION
		   
END_LOOP: MOV AH, 09H
		  MOV DX, OFFSET STRING2
		  INT 21H
		   
		  MOV AH, 4CH
		  INT 21H
CODE ENDS
	 END START
```

## Little Note on Square Brackets

[This SO question](https://stackoverflow.com/questions/48608423/what-do-square-brackets-mean-in-x86-assembly) explains very well on how square brackets work in assembly. In brief, we can say that:

```masm
MOV AX, 1234H
MOV BX, [AX]
```

can be understood as a dereferencing operation such as:

```masm
MOV BX [1234H]
```

We started by copying 1234H into AX register. In `MOV BX, [AX]`, AX holds the value of 1234H, so we can say that this operation copies the value from the address 0x1234H (`[AX] = [1234H]`) onto BX register.

We can see similar operations with square brackets such as copying the value from AX to the address BX is pointed to (`[BX]`):

```masm
MOV [BX] AX
```

Since BX is enclosed with square brackets, the value of the actual BX register remains the same, instead the value is copied to the **address BX points to:**

```
 | AX : 01234567 |   --no-->   | AX : 01234567 |
 | BX : 00000008 | --change--> | BX : 00000008 |

ADDRESS           VALUE
00000000          6A43210D   ->   6A43210D 
00000004          51C9A847   ->   51C9A847 
00000008          169B87F1 =====> 01234567 
0000000C          C981A517   ->   C981A517 
00000010          9A16D875   ->   9A16D875 
00000014          54C9815F   ->   54C9815F 
```

BX holds the value of `00000008`, so it copies to the value stored in address `00000008` in the memory. It's convenient enough to think that values stored in registers and memory are separate.

### Addressing Memory with Registers

Here we try to access the address `[AX]` points to, grab its value and copies it to BX register:

```masm 
MOV BX, [AX]
```

Right now we're accessing the value stored in the **address of the memory, given by the value of AX:**

```
 | AX : 00000008 |    ->     | AX : 00000008 |
 | BX : 01234567 |   ====>   | BX : 169B87F1 |

[No change to memory]
ADDRESS           VALUE
00000000          6A43210D
00000004          51C9A847
00000008          169B87F1
0000000C          C981A517
00000010          9A16D875
00000014          54C9815F  
```

It is important to determine which location of the value u want to copy from, this leads to the value stored in memory address or the register, and vice versa.

## String Operations

### Often Used Registers (SI, DI, CX)

Strings in ASM are stored as a sequence of bytes / words in consecutive memory locations. ASM provides instructions designed to operate on strings that can be directly accessed. These are the Source Index (SI) and Destination Index (DI).

SI holds the source address for operations involving **moving data (second operand), loading data, and scanning data.** It is convenient to think of SI being the source that u want to retrieve a string from.
 
DI indexes the destination of the array for **storing data, and moving data (first operand) operations.** We can also conveniently say that DI points to the destination u want to store data in.

CX will still be a register frequently used since string operations are iterative, we need a counter to keep track of individual character instructions along the string.

### Loop Over Data: `REP` and Variants

ASM provides `REP`, `REPE`, `REPZ`, `REPNZ`, `REPNE` to repeat string instructions. It is usually paired with string operations in a single line that we will explain later. We will introduce them in brief.

**All repeat operations depend on CX to count iterations,** so we need to load the number of iterations we want to repeat into CX register.

- `REP`: Repeat
    - ends if `CX == 0`
- `REPE` / `REPZ`: Repeat while equal / zero
    - ends if `ZF == 0` or `CX == 0`, otherwise loop
- `REPNE` / `REPNZ`: Repeat while not equal / zero
    - ends if `ZF == 1` or `CX == 0`, otherwise loop

### Instructions

**Copying a String**

- `MOVSB` / `MOVSW`: Move bytes / words from source to destination

This instruction is particularly useful for copying strings from one source to a destination. We will be using ES (Extra Segment) to declare another size 13 empty array with `DB 13 DUP(?)`.

```masm
DATA SEGMENT
SOURCE DB 'Hello, World!', "$"  ; Source string
DATA ENDS

EXTRA SEGMENT
DEST DB 13 DUP(?) ; empty array with size 13
EXTRA ENDS

CODE SEGMENT
    ASSUME CS:CODE, DS:DATA, ES:EXTRA
START:
    MOV AX, DATA
    MOV DS, AX                   
    MOV AX, EXTRA
    MOV ES, AX 

    CLD ; clear direction flag (increment mode)
    LEA SI, SOURCE               
    LEA DI, DEST   
    MOV CX, 13                   
    REP MOVSB ; Copy bytes from DS:SI to ES:DI
	
    MOV BYTE PTR [DI], "$" ; sets the byte at DI to "$"
    LEA DX, DEST
    MOV AH, 09H
    INT 21H
	
    MOV AH, 4CH                ; Terminate program
    INT 21H
CODE ENDS
    END START
```

**Storing Characters into Buffer**

- `STOSB` / `STOSW`: Store a byte / word into memory location

We say that a buffer is a temporary location to hold data in the memory. Here we use an array as the data strucutre for buffer. This instruction is mainly used to fill a buffer with characters.

- `LODSB` / `LODSW`: Load a byte / word into AX / AL

Once we stored data into the buffer, we can use this instruction to load these characters into AL for further usage.

```masm
DATA SEGMENT
BUFFER DB 20 DUP(?)
DATA ENDS

CODE SEGMENT
ASSUME CS:CODE DS:DATA ES:DATA
START: MOV AX, DATA
	   MOV DS, AX
	   MOV ES, AX
	   
	   LEA DI, BUFFER ; point DI to buffer
	   MOV AL, "#"
	   MOV CX, 20
	   
	   CLD
	   REP STOSB ; store character "#" from AL 20 times
	   
	   MOV CX, 20
	   LEA SI, BUFFER ; point SI to buffer
	   
PRINT: LODSB ; load byte at DS:SI into AL and increment SI
	   MOV DL, AL
	   MOV AH, 02H
	   INT 21H
	   LOOP PRINT
	   
	   MOV AH, 4CH
	   INT 21H
CODE ENDS
	 END START
```

**Scaning a String**

- `SCASB` / `SCASW`: Scan a byte / word

This instruction is particularly useful for checking if a character is in a string:

```masm
DATA SEGMENT
STRING1 DB "Hello world!", "$"
STRING2 DB "Character not found", "$"
STRING3 DB "Character found", "$"
DATA ENDS

EXTRA SEGMENT
CHAR DB "W"
EXTRA ENDS

CODE SEGMENT
ASSUME CS:CODE, DS:DATA, ES:EXTRA
START: MOV AX, DATA
	   MOV DS, AX
	   MOV AX, EXTRA
	   MOV ES, AX
	   
	   LEA DI, STRING1
	   MOV AL, CHAR
	   MOV CX, 13
	   
	   CLD
	   REPNE SCASB
	   
	   JNZ NOT_FOUND
	   
	   LEA DX, STRING3
	   MOV AH, 09H
	   INT 21H
	   JMP PROGRAM_END
	   
NOT_FOUND: LEA DX, STRING2
		   MOV AH, 09H
		   INT 21H
		   
PROGRAM_END: MOV AH, 4CH
			 INT 21H
CODE ENDS
	 END START
```

## Pointers

Recall that we use pointers to access memory addresses, different pointers tell the machine the type, size and location of the data being referenced.

### Size Pointers

This type of pointers indicate a specific type of value we are working on which include byte, word and double word:

- `BYTE PTR`
    - explicitly tell the assembler the operation is accessing a byte-sized value (8 bits)

- `WORD PTR`
    - accessing word-sized value (16 bits), such as storing and moving a word in memory.

- `DWORD PTR`
    - double word (32 bits) data

### Intra / Inter Segment Access Pointers (Subroutines)

Pointers can access memory within the same segment (Intra) or across different segment (Inter). It is typically useful in subroutine jump and call instructions.

- `FAR PTR`
    - access **inter segment** subroutines
    - composed of a segment and offset (address)

```masm
JMP FAR PTR [BX] ; Jump to a far address pointed to by BX (segment:offset)
```

- `NEAR PTR`
    - access **intra segment** subroutines
    - just an offset (without segment)

```masm
CALL NEAR PTR [SI] ; call a subroutine at the address pointed by SI (offset)
```

## More examples

### Turning a String to Lowercase

```masm
DATA SEGMENT
SOURCE DB "HELLOWORLD"
DATA ENDS

EXTRA SEGMENT
DEST DB 10 DUP(?)
EXTRA ENDS

CODE SEGMENT
ASSUME CS:CODE DS:DATA ES:EXTRA
START: MOV AX, DATA
	   MOV DS, AX
	   MOV AX, EXTRA
	   MOV ES, AX
	   
	   CLD
	   LEA SI, SOURCE
	   LEA DI, ES:DEST
	   MOV CX, 10
	   
AGAIN: ADD BYTE PTR [SI], 20H 
	   ; uses SI to point to the index of the string and adds 20H
	   ; essentially making the character lowercase
	   MOVSB
	   DEC CX
	   JNZ AGAIN
	   
	   ; optional
	   CLD
	   MOV CX, 10
	   LEA SI, ES:DEST
	   
PRINT: LODSB
	   MOV DL, AL
	   MOV AH, 02H
	   INT 21H
	   LOOP PRINT
	   
	   MOV AH, 4CH
	   INT 21H
CODE ENDS
	 END START
```

### Count Frequency of Character in String

```masm
DATA SEGMENT
STRING DB "HELLOWORLD", "$"
RESULT DW 0
DATA ENDS

CODE SEGMENT
ASSUME CS:CODE DS:DATA ES:EXTRA
START: MOV AX, DATA
	   MOV DS, AX
	   
	   LEA SI, STRING
	   MOV AL, "L"
	   
	   CLD
	   MOV CX, 10
	   MOV BX, 0
	   
CHECK: CMP BYTE PTR [SI+BX], AL
	   JNZ SKIP
	   INC RESULT
	   
SKIP: INC BX
	  DEC CX
	  JMP CHECK

OVER: MOV AH, 4CH
	  INT 21H
CODE ENDS
	 END START	   
```

## Stack Segment and Related Operations

We begin to discuss Stack Segment (SS) and Stack Pointer (SP) as well as common instructions used. Recall that a stack is a LIFO data structure with the top element, push and pop operations.

- SS points to the segment in memory used for stack
- SP points to the current top of the stack

Note that the stack grows downward, **pushing elements requires `SP` to decrement 2 bytes while popping increments 2 bytes.**

Here's an example of pushing items 12 and 34 from AX with SP initially pointing to address 2000H.

![image](https://hackmd.io/_uploads/rkIxUy571g.png)

### Pushing and Popping

- `PUSH`: pushes registers, addresses, or values to stack
    - decrements `SP` by 2 bytes

```masm
MOV AX, 25
PUSH AX
```

- `POP`: removes data from the stack
    - increments `SP` by 2 bytes

```masm
POP BX
```

### Usages

Stack is generally essential for procedures and subroutines for **saving the return address and passing parameters.**

**Call Subroutine**

When a `CALL` instruction executes to call a subroutine, the return address is pushed onto the stack. When there's a `RET` return instruction, the return address gets popped off the stack and used to continue exectuion/

```masm
CALL PROCEDURE ; calls procedure, pushes return address to stack
...
PROCEDURE: ...
           RET ; return to caller, pops return address from stack
```

**Save Registers within Subroutine**

Stack can be used to save registers at the beginning of a subroutine and restore them before returning to ensure the registers will not be overwritten.

```masm
PROCEDURE: PUSH AX
           PUSH BX
           ...
           POP AX
           POP BX
           RET
```

This operation saves the values from AX and BX via pushing to stack. The rest of the program continues until popping AX and BX values, returning to the caller.

**Passing Parameters onto The Stack**

We can also pass **parameters of subroutine** onto the stack. Caller pushes arguments onto the stack and saves them for later via SP.

## Shift and Rotate
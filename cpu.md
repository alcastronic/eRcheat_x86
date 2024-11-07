# Registers

## General Purpose Registers

Source: [x86re.com](https://x86re.com/1.html)

```
    ┌───────────────┬───────┬───────┐
    │        16 bits│ 8 bits│ 8 bits│
    ┌───────────────┬───────┬───────┐ ──┐
EAX │            AX │   AH  │   AL  │   │
    ├───────────────┼───────┼───────┤ D │
EBX │            BX │   BH  │   BL  │ A │
    ├───────────────┼───────┼───────┤ T │
ECX │            CX │   CH  │   CL  │ A │
    ├───────────────┼───────┼───────┤   │
EDX │            DX │   DH  │   DL  │   │
    ├───────────────┴───────┴───────┤  ─┤
ESI │                               │ I │
    ├───────────────────────────────┤ N │
EDI │                               │ D │
    ├───────────────────────────────┤ / │
ESP │                               │ P │
    ├───────────────────────────────┤ N │
EBP │                               │ T │
    └───────────────────────────────┘ ──┘
    │                        32 Bits│
    └───────────────────────────────┘
```

Data Registers:
* EAX: Accumulator, often for math and return values.
* EBX: Base, often holds function params and indexed addresses.
* ECX: Count, for loop counts (programmers: this is "i") and function params.
* EDX: Data, general use and function params.
* Internally labeled boxes are subregisters which can be used.

Index Registers
* ESI: Source Index, often stores constants or used as a pointer.
* EDI: Destination Index, often a destination for data or used as a pointer.

Pointers
* Points to a memory location, generally expressed in hex. Example: 0x40154b.
* ESP: Stack pointer, stores a pointer to top of stack.
* EBP: Base pointer, stores a pointer to base of current stack frame.


## Flags


Certain instructions change flags, which are represented as simple 1 or 0 on a flag register.
For example, if we perform an "add" between two numbers, we may have a few outcomes that we need
to be aware of. Let's say we compare two values to see if they are equal with CMP EDX, EAX. The
"zero flag" will either be set to 1 if they are equal, or 0 if they are not. An instruction
like JZ 0x40154b that jumps to memory location 0x40154b if the comparison equaled to zero would
check this flag. Think of these as boolean variables.

```
┌───────────────────────────────────────────────────────────────────────────┐
│ CMP    EAX, EDX    ; Changes zero flag (ZF) to 1 if EAX and EDX not equal │
│ JNE    0x40154b    ; If ZF set to 1, jump to 0x40154b                     │
└───────────────────────────────────────────────────────────────────────────┘
```

There are a handful of flags to learn, but a few to get you started are:
* `ZF`, `zero` flag, set if result is 0.
* `SF`, `sign` flag, set if a result is negative.
* `OF`, `overflow` flag, set if a result was larger than a register can hold. Let us say AL and DL
    are set to 0xFF. ADD AL, DL would return a 9 bit character and therefore overflow the 8 bit
    register.


## Segment Registers

In the x86 architecture, segment registers are used to hold the segment base addresses for different segments of memory. These registers are essential for the segmented memory model used in x86 processors. Here are the primary segment registers in x86:

1. **CS (Code Segment)**:
   - Holds the base address of the code segment, where the executable instructions are located.
   - The instruction pointer (IP or EIP) points to the next instruction to be executed within this segment.

2. **DS (Data Segment)**:
   - Holds the base address of the data segment, where global and static variables are stored.
   - Used for accessing data variables.

3. **SS (Stack Segment)**:
   - Holds the base address of the stack segment, where the stack is located.
   - The stack pointer (SP or ESP) points to the top of the stack within this segment.

4. **ES (Extra Segment)**:
   - An additional data segment register used for string operations and other data accesses.
   - Often used with the `SI` (Source Index) and `DI` (Destination Index) registers for string manipulation instructions.

5. **FS and GS**:
   - Additional segment registers introduced in later x86 processors.
   - Often used for special purposes, such as thread-local storage or accessing specific data structures in operating systems.

### Summary of Segment Registers

- **CS**: Code Segment
- **DS**: Data Segment
- **SS**: Stack Segment
- **ES**: Extra Segment
- **FS**: Additional Segment (often used for thread-local storage)
- **GS**: Additional Segment (often used for special purposes)

These segment registers are used with the offset registers (such as EIP, ESP, EBP, ESI, and EDI) to form a complete address in the segmented memory model.

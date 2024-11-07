# Anti Disassembly

##  Polymorphic Malware

Simple technique to archive ever-changing signatures (in each build), encrypt the code, use a different key each time.

Example decryption:

```asm
mov ebx, size
mov edi, start_addr
xor cs:[edi], 97
inc edi
dec ebx
jnz 0x106
jmp code_start
```

## Self modifying code

The program writes data into the own code section.

```asm
0x100: call 0x103               ; pushes eip to stack
0x103: pop eax                  ; load eip (0x103)
0x104: sub eax, 3               ; eax now back to 0x100
0x107: add cs:[0x101], 0x10     ; add 0x10 to the instruction located at 0x101
0x10d: jmp eax                  ; jmp to the modified instruction

...is changed to...

0x100: call 0x113               ; modified instruction is now 0x113
```

*Note:* In modern Windows OS this is not possible like the above instruction suggests, but may be archived
with the following flow (not part of our lecture):

This requires it to either change protections several times, using `VirtualProtect` for instance.

```c
#include <windows.h>

void modify_code() {
    DWORD oldProtect;
    BYTE* code = (BYTE*)0x00401000; // Address of the code to modify

    // Change the protection to PAGE_EXECUTE_READWRITE
    VirtualProtect(code, size, PAGE_EXECUTE_READWRITE, &oldProtect);

    // Modify the code
    code[0] = 0x90; // NOP instruction

    // Restore the original protection
    VirtualProtect(code, size, oldProtect, &oldProtect);
}
```

Code Source ChatGPT

## Unaligned Branches

Assembly instructions have different lengths, this makes it possible to "hide" opcodes in longer opcodes.
By jumping into the opcode, at a defined offset, it is possible to call the "hidden" opcode.

```asm
0x100: mov ax, 0x3eb    ; long opcode with "hidden" opcode
jmp 0x101               ; jump to "hidden" opcode
call 0xf7e9             ; this might change
```

Tipp: In IDA type `U` to undefine code at `0x100`, then press `C` at `0x101` to define code.

The above code resolves to the following:

```asm
0x101: eb 03 jmp 0x106
0x103: eb fc jmp 0x101
0x105: 9a e9 f7 00 01 call 0x100:0xf7e9 ; this is also contains hidden codejmp addr
```

## Control Flow Obfuscation

Overcomplicate the control flow.

```asm
jmp addr            ; jmp to addr
```

This can be complicated:

```asm
0x100: push addr
0x103: call 0x106
0x106: pop ax
0x107: jmp ax       ; jmp to addr
```

It can be further complicated:

```asm
0x100: push addr
0x103: call 0x106
0x106: pop ax       ; store return address in ax
0x107: jmp 0x10a    ; jmp
0x109: nop
0x10a: add ax, 4    ; 1. 0x106+4 = 0x10A, 2. 0x10A + 0x4 = 0x10E
0x10d: push ax      ;
0x10e: ret          ; 1. ret 0x10A, ret 0x10E, ret addr
```

Both examples misuse `ret` as `jmp`.

## isDebugger present

Either call library function is debugger present or [get value from PEB](https://reverseengineering.stackexchange.com/questions/6024/how-does-this-test-for-debugger).

The code is basically what IsDebuggerPresent does.

1. Get a pointer to the TEB (located at fs:18h)
2. Get a pointer to the PEB (located at teb+30h)
3. Check the `BeingDebugged` flag (located at peb+2)


```asm
pop     eax
mov     eax, large fs:18h   ; Get adddress from TIB at fs:18h
mov     eax, [eax+30h]      ; Access PEB address in TIB https://learn.microsoft.com/en-us/windows/win32/api/winternl/ns-winternl-teb
mov     bl, [eax+2]         ; Second argument in PEB is BeingDebugged flag https://learn.microsoft.com/en-us/windows/win32/api/winternl/ns-winternl-peb
mov     var_global_dbg, bl  ; Store return value somewhere
retn
```

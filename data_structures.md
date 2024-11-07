
# Data Structures

## Variables

### Global

Variable address is known at compile time, can be addressed directly

```asm
mov eax, [0x00402030]
```

### Local

```asm
mov eax, [ebp-0x20] ; negative offset to ebp
```

### DLL imported

Variable addressed in imported DLL

```
mov eax, [IAT_ENTRY_Variable1]
```

### Thread

Thread variables which are stored in `Thread Local Storage` are accessed via `TlsGetValue` and `TlsSetValue`.

## Elementary Data types

The used elementary datatype should be clear from context.

## Arrays

### Local Array

Element adrress = [`Start address of array` + `Index` * `element size`]

```c
void main() {
    WORD LocalArray[10];
    DWORD i = 6;
    LocalArray[0] = 0x10;
    LocalArray[1] = 0x20;
    LocalArray[2] = 0x30;
    LocalArray[i] = 0x60;
}
```

```asm
fun_main:
    push ebp                            ; save old base pointer to stack
    mov ebp, esp                        ; create new base pointer
    sub esp, 18h                        ; substract 24 Byte, 20 Byte for LocalArray and 4 for a local variable (i)
    mov dword ptr [ebp-0x18], 6         ; stored 6 at the offset from ebp "ebp-0x18" in variable "i"
    mov dword ptr [ebp-0x14], 10h       ; stored 0x10 at the negative offset from ebp "ebp-0x14"
                                        ; the addressing is constant
    mov dword ptr [ebp-0x12], 20h       ; ...
    mov dword ptr [ebp-0x10], 30h       ; ...
    mov eax, [ebp-0x18]                 ; move the value of i to eax
    mov word ptr [ebp-0x14+eax*2], 60h  ; address the array with the calculated offset address of:
                                        ; array_start = 0x14; index = 6 and element_size = 2 Byte
                                        ; store the value 0x60 at this index
    mov esp, ebp                        ; restore old ebp
    pop ebp
    retn                                ; return to return address
```

This produces the following stack frame, after the function is called.

```c
 -------------------------------
|  ebp-0x18 |   i               |
|  ebp-0x14 |   LocalArray[0]   |
|  ...      |   ...             |
|  ebp-0x2  |   LocalArray[9]   |
|  ebp      |   old ebp         |
 -------------------------------
```

### Global Array

```c
void main() {
    DWORD i = 6;
    GLoabalArray[0] = 0x10;
    GLoabalArray[1] = 0x20;
    GLoabalArray[2] = 0x30;
    GLoabalArray[i] = 0x60;
}
```

```asm
fun_main:
    push ebp                            ; save old base pointer to stack
    mov ebp, esp                        ; create new base pointer
    sub esp, 4h                         ; substract 4 Bytes for a local variable (i)
    mov dword ptr [ebp-0x4], 6          ; stored 6 at the offset from ebp "ebp-0x4" in variable "i"
    mov word_406120, 10h                ; stored 0x10 at the address of the global variable
                                        ; the addressing is constant "variabel start + index offset"
    mov word_406122, 20h                ; ...
    mov word_406124, 30h                ; ...
    mov eax, [ebp-0x4]                  ; move the value of i to eax
    mov word_406120[eax*2], 60h         ; address the global array with the glbal_var_address = word_406120;
                                        ; and the index calculation with index = 6 and element_size = 2 Byte
                                        ; store the value 0x60 at this index
    mov esp, ebp                        ; restore old ebp
    pop ebp
    retn                                ; return to return address
```

This produces the following stack frame, after the function is called.

```c
 -------------------------------
|  ebp-0x4  |   i               |
|  ebp      |   old ebp         |
 -------------------------------
```

## Two-dimensional array


```c
void fun_main() {
    int LocalArray[4][4]; // 4x4 integer array
    int i = 6;            // local variable i
    int j = 2;            // assume j = 2

    // Initialize the array with specific values
    LocalArray[0][0] = 0x10;
    LocalArray[0][1] = 0x20;
    LocalArray[0][2] = 0x30;
    LocalArray[0][3] = 0x40;
    LocalArray[1][0] = 0x50;
    LocalArray[1][1] = 0x60;
    LocalArray[1][2] = 0x70;
    LocalArray[1][3] = 0x80;
    LocalArray[2][0] = 0x90;
    LocalArray[2][1] = 0xA0;
    LocalArray[2][2] = 0xB0;
    LocalArray[2][3] = 0xC0;
    LocalArray[3][0] = 0xD0;
    LocalArray[3][1] = 0xE0;
    LocalArray[3][2] = 0xF0;
    LocalArray[3][3] = 0x100;

    // Calculate the address of LocalArray[i][j]
    LocalArray[i][j] = 0x60; // Store the value 0x60 at LocalArray[i][j]
}

int main() {
    fun_main();
    return 0;
}

```

```asm
fun_main:
    push ebp                            ; save old base pointer to stack
    mov ebp, esp                        ; create new base pointer
    sub esp, 40h                        ; substract 64 bytes for LocalArray (4x4 integers) and 4 bytes for a local variable (i)
    mov dword ptr [ebp-0x40], 6         ; store 6 at the offset from ebp "ebp-0x40" in variable "i"
    mov dword ptr [ebp-0x3C], 10h       ; store 0x10 at the negative offset from ebp "ebp-0x3C"
    mov dword ptr [ebp-0x38], 20h       ; store 0x20 at the negative offset from ebp "ebp-0x38"
    mov dword ptr [ebp-0x34], 30h       ; store 0x30 at the negative offset from ebp "ebp-0x34"
    mov dword ptr [ebp-0x30], 40h       ; store 0x40 at the negative offset from ebp "ebp-0x30"
    mov dword ptr [ebp-0x2C], 50h       ; store 0x50 at the negative offset from ebp "ebp-0x2C"
    mov dword ptr [ebp-0x28], 60h       ; store 0x60 at the negative offset from ebp "ebp-0x28"
    mov dword ptr [ebp-0x24], 70h       ; store 0x70 at the negative offset from ebp "ebp-0x24"
    mov dword ptr [ebp-0x20], 80h       ; store 0x80 at the negative offset from ebp "ebp-0x20"
    mov dword ptr [ebp-0x1C], 90h       ; store 0x90 at the negative offset from ebp "ebp-0x1C"
    mov dword ptr [ebp-0x18], 0A0h      ; store 0xA0 at the negative offset from ebp "ebp-0x18"
    mov dword ptr [ebp-0x14], 0B0h      ; store 0xB0 at the negative offset from ebp "ebp-0x14"
    mov dword ptr [ebp-0x10], 0C0h      ; store 0xC0 at the negative offset from ebp "ebp-0x10"
    mov dword ptr [ebp-0x0C], 0D0h      ; store 0xD0 at the negative offset from ebp "ebp-0x0C"
    mov dword ptr [ebp-0x08], 0E0h      ; store 0xE0 at the negative offset from ebp "ebp-0x08"
    mov dword ptr [ebp-0x04], 0F0h      ; store 0xF0 at the negative offset from ebp "ebp-0x04"
    mov dword ptr [ebp], 100h           ; store 0x100 at the offset from ebp "ebp"

    mov eax, [ebp-0x40]                 ; move the value of i to eax (i = 6)
    mov ecx, 2                          ; assume j = 2
    mov edx, 4                          ; number of columns
    imul eax, edx                       ; eax = i * number of columns
    add eax, ecx                        ; eax = i * number of columns + j
    shl eax, 2                          ; eax = (i * number of columns + j) * 4 (element size)
    mov dword ptr [ebp-0x3C+eax], 0x60  ; store the value 0x60 at LocalArray[i][j]

    mov esp, ebp                        ; restore old ebp
    pop ebp
    retn                                ; return to return address
```


Stack Layout

```c
 -------------------------------
|  ebp+0x??  |   return address  | <- return address pushed by the call instruction
 -------------------------------
|  ebp       |   old ebp         | <- saved base pointer
 -------------------------------
|  ebp-0x04  |   LocalArray[3][3]| <- start of LocalArray
|  ebp-0x08  |   LocalArray[3][2]|
|  ebp-0x0C  |   LocalArray[3][1]|
|  ebp-0x10  |   LocalArray[3][0]|
|  ebp-0x14  |   LocalArray[2][3]|
|  ebp-0x18  |   LocalArray[2][2]|
|  ebp-0x1C  |   LocalArray[2][1]|
|  ebp-0x20  |   LocalArray[2][0]|
|  ebp-0x24  |   LocalArray[1][3]|
|  ebp-0x28  |   LocalArray[1][2]|
|  ebp-0x2C  |   LocalArray[1][1]|
|  ebp-0x30  |   LocalArray[1][0]|
|  ebp-0x34  |   LocalArray[0][3]|
|  ebp-0x38  |   LocalArray[0][2]|
|  ebp-0x3C  |   LocalArray[0][1]|
|  ebp-0x40  |   LocalArray[0][0]|
|  ebp-0x44  |   i               | <- local variable i
|  ebp-0x48  |   j               | <- local variable j
 -------------------------------
```

## Struct



## Union
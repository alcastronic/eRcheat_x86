# Control Flow

## Conditional statements & Branching

```C
if (SomeVar == 0) {
    function1();
}
```

```asm
mov eax, [SomeVar]
test eax, eax ; set zeroflag if result is zero
jnz lab_end   ; take jump to end when zeroflag is not set
call function1
```

### Single branch

```c
if( SomeVar == 1 ) {
    function1();
} else {
    function2();
}
```

```asm
    cmp [SomeVar], 1 ; compare var to the value
    jne lab_else     ; if the condition does not match jump to lab_else
    call function1
    jmp lab_end
lab_else:
    call function2
lab_end:
```

### Several banches

```c
if( SomeVar == 1 ) {
    function1();
} else if (SomeVar == 2) {
    function2();
} else {
    function3();
}
```

```asm
    cmp [SomeVar], 1 ; compare var to the value
    jne lab_else1    ; if the condition does not match jump to lab_else1
    call function1
    jmp lab_end
lab_else1:
    cmp [SomeVar],2  ; do another comparison
    jne lab_else2    ; if condition does not match jump to lab_else2
    call function2
    jmp lab_end
lab_end:
```

### Composed condition

```C
if (Var1 == 1 && Var2 < 3) {
    function1();
}
```

```asm
    cmp [Var1], 1
    jne lab_end     ; take jump to end when condition is not met
    cmp [Var2], 3
    jae lab_end     ; take jump to end when condition is not met
    call function1
lab_end:
```

## Switch Case


```c
switch( Var1 ) {
    case 0: RC = 0 ; break;
    case 1: RC = 1  ; break;
    case 2: RC = 2  ; break;
    case 3: RC = 3  ; break;
    default: RC = 0 ; break
}
```

Produces a branching structure and a switch table

Structure in code section:

```asm
    mov ecx, [Var1]         ; else store Var1 in ecx
    jmp switch_table[ecx*4] ; jump to switch table at offset
                            ; if value of Var1 == 1 --> 1*4
lab_case0:
    mov [RC], 0
    jmp lab_end
lab_case1:                  ; we jump here
    mov [RC], 1
    jmp lab_end
    ...
lab_default:
    mov [RC], 0
    jmp lab_end
lab_end
```

switch table in data section

```asm
switch_table:
0x00 dd offset lab_case0
0x04 dd offset lab_case1 ; we take this address (switch_table[1*4])
0x08 dd offset lab_case2
0x0C dd offset lab_case3
```

## Binary Tree

```asm
switch( Var1 )
{
    case 0x0:  ...
    case 0x100:  ...
    case 0x150:  ...
    case 0x172:  ...
    case 0x210:  ...
    case 0x230:  ...
    case 0x2A0:  ...
    default:  ...
}
```

```asm
cmp [Var1], 0x172
jg lab_above_0x172
cmp [Var1], 0x172
je lab_0x172

cmp [Var1], 0x100
jg lab_above_0x100
cmp [Var1], 0x100
je lab_0x100

cmp [Var1], 0x0
je lab_0x0

jmp lab_default

lab_above_0x100:
cmp [Var1], 0x150
je lab_0x150
jmp lab_default

lab_above_0x172:  ...
```

## Loops

### While

```c
c = 0;
while( c < 0x100) {
    function1();
    c++;
}
```

```asm
    mov ecx, 0
lab_start:
    cmp ecx, 0x100
    jae lab_exit
    call function1
    inc ecx
    jmp lab_start
lab_exit:
```

### Do-While

```c
c = 0;
do {
    function1();
    c++;
} while( c < 0x100);
```

```asm
    mov ecx, 0
lab_start:
    call function1
    inc ecx
    cmp ecx, 0x100
    jl lab_start
```

### While with continue and break

```c
c = 0;
while( TRUE) {
    c++
    if ( c == 3 ) {
        continue;
    }
    if (c == 5) {
        break;
    }
    function1();
}
```

```asm
    mov ecx, 0
lab_start:
    inc ecx
    cmp ecx, 3
    jnz lab_not3    ; when not three jump to next if
    jmp lab_start   ; continue statement
lab_not3:
    cmp ecx, 5
    je lab_exit     ; when condition is met, break
    call function1
    jmp lab_start
lab_exit
```

### For

```c
int c = 3;
for ( int i = 0; i <c; i++ ) {
    function1();
}
```

```asm
    mov [var_c], 3
    mov [var_i], 0
loop:
    mov eax, [var_i]
    inc eax
    mov [var_i], eax
after_inc:
    mov eax, [var_i]
    cmp eax, [var_c]
    jge exit_lopp
    call function1
    jmp loop
exit_loop:
```

## Function calling

Some bits are from [x86re.com/1.html](https://x86re.com/1.html)

Example according to `stdcall` convention.

```c
x = func1(y,4)

stdcall
int func1(int a, int b) {
    int z;
    z = a * b;
    return z;
}
```

```c
 -----------------------
|   call stack          |
 -----------------------
|  ebp-0x4 |   z        |
|      ebp |   old ebp  |
|  ebp+0x4 |   RET      |
|  ebp+0x8 |   a        |
|  ebp+0xc |   b        |
 -----------------------
```

Call the function

```asm
    push 4              ; 2nd argument
    push [var_y]        ; 1st argument
    call func1          ; call function -> pushes return address to stack
    mov [var_x], eax    ; store the result upon return
```

Function declaration

Prolog:

```asm
func1:
    push ebp        ; save old base pointer to stack
    mov ebp, esp    ; create new base pointer
    sub esp, 4      ; substract 4 Bytes for a local variable (z)
```

Function body

```asm
    mov ecx, [ebp+0x8]      ; address the 2nd passed function parameter "ebp+0x8"
    imul ecx, [ebp+0xC]     ; address the 1st passed function parameter "ebp+0xC"
    mov [ebp-4], ecx        ; address the local variable "ebp-4"
```

Store the return value

```asm
    mov eax, [ebp-4]        ; store result of var "c" in eax to make it available to the calling function
```

Prolog

```asm
    mov esp, ebp    ; restore old ebp
    pop ebp         ; esp now point to return address (could also be leave)
    ret 8           ; return to return address and remove passed function parameters (8 Byte)
                    ; an alternative to ret 8 is to "add esp, 0x8" before "pop ebp; ret".
```

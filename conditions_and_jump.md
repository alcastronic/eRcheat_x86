# Conditions & Jumps

## Conditions

### TEST

Does bitwise logical and.

```asm
test eax, eax   ; eax AND eax
```

Flags:

- `SF`
- `ZF` = 1 if bitwise `op1 AND op2` is zero.
- `PF`

### CMP

Subtracts the second operand from the first one.

```asm
cmp rax, rbx    ; rax - rbx
```

- `ZF` = 1 if `op1 == op2` else `ZF` = 0
- `SF` = 1 if `op1 < op2` else `SF` = 0


## Jumps

```c
                        Do jump when flag has this value
                                    \      \
------------------------------------------------
|   Letter 	 |   Meaning         |  ZF  |  SF  |
------------------------------------------------
|    j       |   (prefix) jump   |      |      |
|    n 	     |   not             |  0   |      |
|    nz	     |   not Zero        |  0   |      |
|    ne      |   not equals      |  0   |      |
|    z 	     |   zero            |  1   |      |
|    e       |   equals          |  1   |      |
|    g       |   greater than    |      |      |
|    l       | 	 less than       |      |      |
|    s       |   sign            |      |  1   |
------------------------------------------------
```

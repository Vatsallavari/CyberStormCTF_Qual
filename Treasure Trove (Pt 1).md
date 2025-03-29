# 🔐 Reverse Engineering Challenge – `trove`

## 📁 Challenge Overview

The `trove` binary is a reverse engineering challenge that prompts for a "secret" string. If the correct input is provided, it decrypts and prints a flag. Rather than reverse-engineering the input logic or brute-forcing the secret, we bypassed the check through static binary patching using Ghidra.

---

## 🛠 Tools Used

- [Ghidra](https://ghidra-sre.org/) for disassembly and patching
- Linux terminal to execute the patched binary

---

## 🔍 Initial Analysis

Upon loading the binary into Ghidra, we observed the following logic in `main()`:

```c
printf("Enter the secret: ");
scanf("%s", local_e8);

if (strncmp(local_e8, local_75, 12) != 0) {
    puts("Incorrect, try again!");
    return 0;
}

printf("Flag: %s\n", local_a8);
```
The check compares user input to a value stored in local_75, which is dynamically generated at runtime. If it doesn't match, the program prints an error and exits.

✂️ Patching Strategy
We found the assembly responsible for the comparison at:

```asm
00401b77: CMP DL, AL
00401b79: JZ  00401b91
```
✅ Step 1: NOP the comparison
At 0x00401b77, we patched:

```asm
CMP DL, AL
```
to:
```asm
NOP
```
✅ Step 2: Force the jump
At 0x00401b79, we patched:

```asm
JZ 00401b91
```
to:
```asm
JMP 00401b91
```
This guarantees the program always jumps to the success path, regardless of user input.

🚀 Execution
After exporting the patched binary:

```bash
chmod +x trove_patched
./trove_patched
Enter the secret: a
Flag: REDFOX{H00Ps-Lo0p5-1-LoV3-7H3m-@lL!}
```
Any input now results in the flag being printed.

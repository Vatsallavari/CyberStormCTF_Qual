# 🔐 Reverse Engineering Challenge – `trove`

## 📁 Challenge Overview (Treasure Trove (Pt 1)
This one sent Beckman for a loop, multiple loops really. Can you help save Benn's reputation in front of Shanks xor is it really that bad?

---

## 🛠 Tools Used

- [Ghidra](https://ghidra-sre.org/) for disassembly and patching
- Linux terminal to execute the patched binary

---

## 🔍 Initial Analysis

Upon loading the binary into Ghidra, we observed the following logic in `main()`:

```c
  iStack_c = 0;
  while( true ) {
    if (0xb < iStack_c) {
      for (iStack_10 = 0; abStack_a8[iStack_10] != 0; iStack_10 = iStack_10 + 1) {
        abStack_a8[iStack_10] = abStack_b4[0] ^ abStack_a8[iStack_10];
      }
      printf(&UNK_0049305b,abStack_a8);
      return 0;
    }
    if (acStack_e8[iStack_c] != acStack_75[iStack_c]) break;
    iStack_c = iStack_c + 1;
  }
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

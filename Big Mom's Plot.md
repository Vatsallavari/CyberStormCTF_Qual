# 🏴‍☠️ CTF Writeup: Big Mom’s Plot


**Challenge Name: Big Mom's Plot**
**Category: Reverse Engineering**
**File:** `warmup.asm`  
**Host: nc cyberstorm.redfoxsec.com 4444**

---

## 📜 Challenge Description
Big Mom's Plot
Big Mom has been keeping their new war strategies under wraps. Not even intelligence services like Cipher Pol have any clue what's going on.
Can you figure out what is it that they are hiding in case the Straw Hat ever come across their armies (again)?

A mysterious server is running at
```yaml
nc cyberstorm.redfoxsec.com 4444
```

We’re tasked with uncovering Big Mom’s hidden plan. Let’s dig in.

---

## 🕵️‍♂️ Recon: What We’re Given
We’re handed an assembly file (warmup.asm) which, at first glance, seems to print a flag when a tilde (~) character is entered.

But the printed flag is:
```
REDFOX{Obviously_This_Is_Real_Flag}
```
This is an intentional decoy — a honeypot for fast clickers.

---

## 🔍 Assembly Code Review (Summary)

The provided warmup.asm performs the following logic:
```asm
section .bss
    buffer resb 100
section .data
    success_msg db "REDFOX{Obviously_This_Is_Real_Flag}", 0xA
    success_len equ $ - success_msg
section .text
    global _start
    _start:
        mov rax, 0      
        mov rdi, 0      
        mov rsi, buffer 
        mov rdx, 100    
        syscall
       
        mov rsi, buffer 
        mov rcx, rax     
        dec rcx        
    check_loop:
        cmp rcx, 0
        je end_program  
        mov al, [rsi]  
        cmp al, 0x7E   
        je okie     
        inc rsi         
        dec rcx         
        jmp check_loop  
    okie:
        mov rax, 1           
        mov rdi, 1          
        mov rsi, success_msg 
        mov rdx, success_len 
        syscall
    end_program:
        mov rax, 60 
        xor rdi, rdi 
        syscall

```
Reads up to 100 bytes from user input.

Scans the input character-by-character.

If it finds ~ (ASCII 0x7E), it jumps to print a hardcoded success message.

Otherwise, it exits silently.

Python equivalent: For better understanding

```python
import sys

# Simulate a 100-byte input buffer
buffer = sys.stdin.read(100)  # reads up to 100 characters from input

# Convert to byte array and trim to actual input length
buffer = bytearray(buffer.encode())
input_length = len(buffer)

# Skip the trailing newline (0x0A) if present
if input_length > 0 and buffer[-1] == 0x0A:
    buffer = buffer[:-1]

# Simulate the assembly loop to scan for the '~' character (0x7E)
found_tilde = False
for byte in buffer:
    if byte == 0x7E:  # ASCII for '~'
        found_tilde = True
        break

# Conditional response based on presence of '~'
if found_tilde:
    print("REDFOX{Obviously_This_Is_Real_Flag}")  # fake flag
# else:
#     do nothing (exit silently, like the assembly does)

```

---

## 🧠 Insight: This is a Decoy
If you fall for the bait and assume that’s the flag, you’ll submit the fake one.

However, the real flag is only returned when you interact with the live service running at:

```bash
nc cyberstorm.redfoxsec.com 4444
```

Also in conditional response below line is suspicious
```do nothing (exit silently, like the assembly does)```
and this condition
```
if byte == 0x7E:  # ASCII for '~'
        found_tilde = True
```
makes our suspicious clear and we try to send '~' as input just to see response.

---

🧪 Solution Steps
Connect to the server:

```bash
nc cyberstorm.redfoxsec.com 4444
```
Send a single tilde ~:

```
~
```
The server responds with the real flag:

```
REDFOX{SO_W3_Wi1L_jU5T_bR1B3_Th3_Mar!n3s_WHIL3_C0mm1t1nG_W4R_Cr1m3s!!}
```

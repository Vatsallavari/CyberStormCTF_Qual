# MerC — RedFox CTF

**Category:** Reverse Engineering  
**Tags:** `reverse` `binary` `ascii logic` `character loop`  
**File:** `MerC`  
**Host:** ```nc cyberstorm.redfoxsec.com 18000```  

---

## 📜 Challenge Description

> Momo trapped a farmer in a binary after finding out about his alleged disservice to Kozuki Oden.  
> He cries out but no one is there to hear him.  
> Help him plead his case by translating what's being lost in the cyberspace.  

---

## 🧠 TL;DR Solution

We reverse engineered the binary to extract a hidden password assembled via ASCII-based conditional logic. By analyzing the character comparisons, we reconstructed the required input:

```
LIVin9HELl
```

When submitted either locally or via the network challenge server, it returned the correct flag.

---

## 🔍 Reverse Engineering Breakdown

Using tools like `strings`, `objdump`, and static disassembly, we identified the binary does the following:

1. **Initial Prompt**:
    - Prints `"Enter the secret:"`
    - Uses `scanf("%10s")` to capture a string into a buffer.

2. **Secret Construction**:
    - The binary constructs a secret string in memory using hardcoded ASCII values in a loop, checking:
      ```c
      if (i == 0x4c) buf[0] = i; // 'L'
      if (i == 0x49) buf[1] = i; // 'I'
      if (i == 0x56) buf[2] = i; // 'V'
      if (i == 0x69) buf[3] = i; // 'i'
      if (i == 0x6e) buf[4] = i; // 'n'
      ...
      ```
    - The second loop continues:
      ```c
      if (i == 0x39) buf[5] = i; // '9'
      if (i == 0x48) buf[6] = i; // 'H'
      if (i == 0x45) buf[7] = i; // 'E'
      if (i == 0x4c) buf[8] = i; // 'L'
      if (i == 0x6c) buf[9] = i; // 'l'
      ```

3. **Comparison Logic**:
    - After building the buffer, the binary compares each character from user input with the constructed one (10 bytes in total).
    - If all characters match, it prints:
      ```
      Correct: redfox{sample_flag}
      ```
    - Otherwise, it prints:
      ```
      Incorrect, try again!
      ```

---

## 🔐 Extracted Secret

Rebuilding the secret from the hardcoded checks gives:

```text
LIVin9HELl
```

---

## 🧪 Execution 

Local Execution
```bash
$ ./MerC
Enter the secret: LIVin9HELl
Correct: redfox{sample_flag}
```

Remote Execution
```bash
$ nc cyberstorm.redfoxsec.com 18000
LIVin9HELl
REDFOX{M0mon0suk3_S4ma_F0rg1v3_M33!!}
```

---

## 🏁 Final Flag

```
REDFOX{M0mon0suk3_S4ma_F0rg1v3_M33!!}
```

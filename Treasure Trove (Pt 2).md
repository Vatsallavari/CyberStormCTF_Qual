# Treasure Trove (Pt 2) — RedFox CTF

**Category:** Reverse Engineering  
**File:** `medium3`  
**Host:** ```nc cyberstorm.redfoxsec.com 19000```
**Tags:** `reverse` `binary` `C` `modulo` `transform`

---

## 🧠 Challenge Description

> Shanks called Marco for help with this one and Marco almost awakened in his Tori Tori Devil Fruit and **TRANSFORM**ed into a big firebird in the process.  
> Help Marco with this binary so he can cool his head down a bit.

---

## 🕵️‍♂️ TL;DR Solution

We reversed the transformation algorithm used by the binary to validate the password. It applies a deterministic transformation over each character of the input **three times**, and compares the final result with a hardcoded target string. By mathematically inverting this transformation (assuming lowercase-only inputs), we derived the correct password.

---

## 🔍 Analysis

### Binary Behavior

The binary prompts the user for a secret password and applies a custom transform function over the input:

```c
for (int j = 0; j < len; j++) {
    buf[j] = ((((j * 7) ^ 0x5a) & 0xff) + buf[j] - 0x61) % 26 + 'a';
}
```
This loop is run three times, progressively modifying the characters.

The target string (after 3 transformations) is hardcoded:

```nginx
uhwymjeukrvfxqhrnlgztwkqhivtjhepmrqdvb
```

---

## 🧮 Reversing the Transform
Let’s define:

s(i) = (((7 * i) ^ 0x5a) & 0xff) % 26

Each transformation:

```r
T(c, i) = (((c - 'a') + s(i)) % 26) + 'a'
```
Since this is applied three times, the net transformation becomes:

```ini
final = (((original - 'a') + 3 * s(i)) % 26) + 'a'
```
So, to recover the original input character:

```ini
original = ((target[i] - 'a') - 3 * s(i)) % 26 + 'a'
```
Assuming the password contains only lowercase letters, this yields a unique result for each position.

---

## 🧰 Script
Here's the Python script used to solve it:

```python
#!/usr/bin/env python3

TARGET = "uhwymjeukrvfxqhrnlgztwkqhivtjhepmrqdvb"

def compute_shift(i):
    s = ((7 * i) ^ 0x5a) & 0xff
    return s % 26

def candidate_char(i, target_char):
    s = compute_shift(i)
    orig_val = ((ord(target_char) - ord('a')) - 3 * s) % 26
    return chr(orig_val + ord('a'))

def main():
    candidate = "".join(candidate_char(i, TARGET[i]) for i in range(len(TARGET)))
    print("✅ Recovered password:")
    print(candidate)

if __name__ == '__main__':
    main()
```

✅ Output
```bash
$ python3 solve.py
```
✅ Recovered password:
```
koevkkglcapohnvurgcgbrgrjbpcdommawwyru

$ ./medium3
Enter the secret password: koevkkglcapohnvurgcgbrgrjbpcdommawwyru
CORRECT! REDFOX{DEMO_FLAG}

$ nc cyberstorm.redfoxsec.com 19000
koevkkglcapohnvurgcgbrgrjbpcdommawwyru
REDFOX{W0W_Y0u_R34lly_Impress3d_5H@nKs_n0W}
```

---

## 🏁 Final Flag

```css
REDFOX{W0W_Y0u_R34lly_Impress3d_5H@nKs_n0W}
```

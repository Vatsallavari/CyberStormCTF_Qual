# 🏴‍☠️ Treasure Trove (Pt 0) – Reverse Engineering Write-Up

**Challenge Name:** Treasure Trove (Pt 0)  
**File:** `warmup`  
**Category:** Reverse Engineering  
**Tool Used:** Ghidra

---

## 📜 Challenge Description

> Shanks found a trove of binaries with treasure hidden inside them. Shanks' crew was able to find most of the treasure, but was struggling with a few.
> Help Shanks get some dough before he sets out to chase the rush of being an Emperor.

---

## 🔍 Static Analysis with Ghidra

After opening the binary in Ghidra, the `main()` function was immediately readable and very straightforward:

```c
undefined8 main(void)
{
  int local_c;

  printf("Enter the Secret Code: ");
  __isoc99_scanf(&DAT_00492048, &local_c);

  if (local_c == 0x539) {
    printf("Good Work! REDFOX{Th1S_Tr3a5ur3_15_ARRRRs}");
  } else {
    printf("Incorrect!");
  }
  return 0;
}
```

---

🏁 Flag
```
REDFOX{Th1S_Tr3a5ur3_15_ARRRRs}
```

# 🧩 Random XOR Challenge Write-up

## 📌 Overview
This binary generates a value using `rand()` and checks user input with an XOR operation. If the condition is satisfied, it prints the flag.

```
if ((key ^ random) == 0xcafebabe)
```

---

## 🔍 Vulnerability Analysis

The program uses `rand()` but does NOT initialize a seed with `srand()`:

```
random = rand();
```

This means `rand()` will always return the same value every time the program runs.

👉 Therefore, the "random" value is actually predictable and can be obtained via debugging.

---

## 🛠️ Debugging

Using GDB (pwndbg), set a breakpoint after the `rand()` call and inspect the stack:

```
b *main+48
run
x/wx $rbp-0x1c
```

Result:

```
random = 0x6b8b4567
```

---

## 🧠 Exploitation Strategy

Given the condition:

```
(key ^ random) == 0xcafebabe
```

Solve for `key`:

```
key = random ^ 0xcafebabe
```

Calculation:

```
key = 0x6b8b4567 ^ 0xcafebabe
    = 0xa175ffd9
```

---

## 🚀 Final Input

The program uses `scanf("%d")`, so input must be in decimal:

```
2708864985
```

---

## ✅ Result

```
Good!
[flag is printed]
```

---

## 📚 Key Takeaways

- `rand()` without `srand()` → deterministic output  
- Debugger can reveal stack-stored values  
- XOR property:
  ```
  A ^ B = C  →  A = B ^ C
  ```
- No brute-force required  

---

## 💡 Summary

👉 Not truly random — just reverse the XOR with a fixed value.

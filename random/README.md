# random

## 🧠 Overview
This challenge is about understanding how pseudo-random values work in C and how improper usage of `rand()` can lead to predictable results.

The program generates a random value and compares it with user input using an XOR operation.  
If the condition is satisfied, the flag is printed.

---

## 🔍 Source Code Analysis

```c
#include <stdio.h>

int main(){
    unsigned int random;
    random = rand();        // random value!

    unsigned int key=0;
    scanf("%d", &key);

    if( (key ^ random) == 0xcafebabe ){
        printf("Good!\n");
        setregid(getegid(), getegid());
        system("/bin/cat flag");
        return 0;
    }

    printf("Wrong, maybe you should try 2^32 cases.\n");
    return 0;
}
```

### Key Points

- A value is generated using `rand()`:
  ```c
  random = rand();
  ```
- User input is read:
  ```c
  scanf("%d", &key);
  ```
- The condition checked:
  ```
  key ^ random == 0xcafebabe
  ```

---

## 💡 Vulnerability

The program uses `rand()` but does NOT use `srand()` to initialize a seed.

👉 This means `rand()` will always return the same value every time the program runs.

So the "random" value is actually **predictable**.

---

## 🛠️ Debugging

Using GDB (pwndbg), we inspect the value stored after the `rand()` call.

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

## 🔑 Exploit Strategy

The condition is:

```
key ^ random == 0xcafebabe
```

We solve for `key`:

```
key = random ^ 0xcafebabe
```

---

## 🧮 Calculation

```
key = 0x6b8b4567 ^ 0xcafebabe
    = 0xa175ffd9
```

---

## 🔢 Decimal Conversion

Since the program uses `scanf("%d")`, we must input a decimal value:

```
2708864985
```

---

## 🧪 Execution

```
./random
2708864985
```

---

## ⚙️ Execution Flow

```
program starts
      ↓
rand() returns fixed value
      ↓
user inputs key
      ↓
XOR operation performed
      ↓
condition satisfied
      ↓
flag is printed
```

---

## 📌 What I Learned

- `rand()` without `srand()` produces deterministic output
- Debugging can reveal hidden values in memory
- XOR can be reversed using:
  ```
  A ^ B = C  →  A = B ^ C
  ```
- No brute-force needed if logic is understood

---

## 🧾 Summary

By recognizing that `rand()` is not properly seeded, we can extract the fixed "random" value via debugging and reverse the XOR condition to compute the correct input.

This allows us to bypass brute-force and directly obtain the flag.

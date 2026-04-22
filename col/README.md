# col

## 🧠 Overview
This challenge is about understanding how input data can be reinterpreted as integers and how endianness affects memory representation.

The program takes a 20-byte input and treats it as an array of 5 integers (4 bytes each).  
If the sum of those integers equals the target hash value, the flag is printed.

---

## 🔍 Source Code Analysis

```c
unsigned long hashcode = 0x21DD09EC;

unsigned long check_password(const char* p){
    int* ip = (int*)p;
    int i;
    int res = 0;
    for(i = 0; i < 5; i++){
        res += ip[i];
    }
    return res;
}
```

### Key Points

- The input must be exactly 20 bytes:
  ```c
  if(strlen(argv[1]) != 20)
  ```
- The input is cast to an `int*`:
  ```c
  int* ip = (int*)p;
  ```
- This means the 20-byte input is interpreted as:
  ```
  ip[0], ip[1], ip[2], ip[3], ip[4]
  ```
- Each `ip[i]` is a 4-byte integer
- The program checks:
  ```
  ip[0] + ip[1] + ip[2] + ip[3] + ip[4] = 0x21DD09EC
  ```

---

## 💡 Understanding the Cast

```c
int* ip = (int*)p;
```

This does NOT change the data.  
It only changes how the data is interpreted.

- Originally: byte-by-byte (`char*`)
- After cast: 4 bytes at a time (`int*`)

So the 20-byte input becomes:

```
[4 bytes][4 bytes][4 bytes][4 bytes][4 bytes]
```

---

## ⚠️ Endianness (Very Important)

On most systems (x86), integers are stored in **little-endian** format.

### Example:

```text
0x12345678
```

In memory:

```text
\x78\x56\x34\x12
```

👉 Least significant byte comes first.

This means when we construct our payload, we must reverse the byte order.

---

## 🔑 Exploit Strategy

We need:

```
ip[0] + ip[1] + ip[2] + ip[3] + ip[4] = 0x21DD09EC
```

### Step 1: Choose simple values

```
ip[0] = 0x01010101
ip[1] = 0x01010101
ip[2] = 0x01010101
ip[3] = 0x01010101
```

Then:

```
0x01010101 * 4 = 0x04040404
```

### Step 2: Calculate last value

```
ip[4] = 0x21DD09EC - 0x04040404 = 0x1DD905E8
```

---

## 🔄 Convert to Little Endian

```
0x01010101 → \x01\x01\x01\x01
0x1DD905E8 → \xe8\x05\xd9\x1d
```

---

## 🧪 Final Payload (20 bytes)

```
\x01\x01\x01\x01
\x01\x01\x01\x01
\x01\x01\x01\x01
\x01\x01\x01\x01
\xe8\x05\xd9\x1d
```

---

## 🐍 Using Python to Generate Input

Typing raw bytes manually is difficult, so we use Python.

### Why Python?

- Allows precise byte control
- Avoids encoding issues
- Useful for all pwnable problems

---

### Basic Concept

```python
b"\x01\x02\x03"
```

👉 This is a byte string, not a normal string.

---

### Correct Way to Output Raw Bytes

```python
import sys
sys.stdout.buffer.write(b"your_bytes")
```

We use `sys.stdout.buffer.write()` instead of `print()`  
because `print()` does NOT output raw bytes correctly.

---

### Full Exploit Command

```bash
./col $(python3 -c 'import sys; sys.stdout.buffer.write(b"\x01\x01\x01\x01"*4 + b"\xe8\x05\xd9\x1d")')
```

---

## ⚙️ Execution Flow

```
python → generate raw bytes
      ↓
shell substitutes output into argv[1]
      ↓
program reads 20 bytes
      ↓
cast to int*
      ↓
sum matches hash
      ↓
flag is printed
```

---

## 📌 What I Learned

- How `char*` can be reinterpreted as `int*`
- Why endianness matters in memory
- How to construct payloads manually
- How to use Python to generate exact byte sequences
- Why `sys.stdout.buffer.write()` is needed

---

## 🧾 Summary

By crafting a 20-byte input where each 4-byte chunk forms integers that sum to the target hash value, and handling little-endian byte order correctly, we can satisfy the condition and obtain the flag.

Python is used to reliably generate and pass the exact byte sequence as program input.

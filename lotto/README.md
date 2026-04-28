# lotto

## 🧠 Overview
This challenge is about understanding how raw byte input is handled and how flawed comparison logic can be exploited.

The program reads 6 bytes from user input and compares them with randomly generated lotto numbers.

---

## 🔍 Source Code Analysis

Input is read as raw bytes:

```
read(0, submit, 6);
```

Random lotto numbers are generated:

```
lotto[i] = (lotto[i] % 45) + 1;   // 1 ~ 45
```

Matching logic:

```
for(i=0; i<6; i++){
    for(j=0; j<6; j++){
        if(lotto[i] == submit[j]){
            match++;
        }
    }
}
```

---

## ⚠️ Vulnerability

The comparison logic is flawed.

- Each lotto number is compared with all 6 input bytes
- Duplicate matches are counted multiple times

👉 This allows a single matching value to increase `match` multiple times

---

## 💡 Exploit Idea

We send 6 identical characters:

```
!!!!!!
```

ASCII value:

```
'!' = 33
```

Since lotto numbers range from 1 to 45, 33 is a valid value.

---

## 🔥 Exploit Logic

If any lotto number equals 33:

```
lotto[i] == 33
```

Then:

```
match += 6
```

👉 Immediately satisfies:

```
match == 6
```

---

## 🚀 Payload

```
!!!!!!
```

---

## 🧪 Execution

```
nc pwnable.kr 9007
```

Then:

```
1
!!!!!!
```

Repeat if needed (≈12.6% success rate)

---

## ⚙️ Execution Flow

```
user input (!!!!!!)
        ↓
converted to bytes [33,33,33,33,33,33]
        ↓
lotto generates random numbers (1~45)
        ↓
if any number == 33
        ↓
match += 6
        ↓
match == 6
        ↓
flag is printed
```

---

## 📌 What I Learned

- `read()` reads raw bytes, not strings
- ASCII values directly affect comparison
- Logical bugs can be more powerful than memory bugs
- Duplicate comparisons can break intended logic

---

## 🧾 Summary

By sending six identical characters (`!!!!!!`), we exploit the flawed nested loop logic.

If just one lotto number matches the value (33), it is counted multiple times, allowing us to satisfy the winning condition easily.

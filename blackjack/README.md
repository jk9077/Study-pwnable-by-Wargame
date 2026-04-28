# blackjack

## 🧠 Overview
This challenge is about identifying logical vulnerabilities in a program rather than playing the game as intended.

The program simulates a Blackjack game, and the goal is to trigger a condition where the player's cash exceeds a certain value to print the flag.

---

## 🔍 Source Code Analysis

Key condition for printing the flag:

```
if (cash > 1000000)
```

Relevant code:

```c
void cash_test()
{
    if (cash <= 0)
    {
        printf("You Are Bankrupt. Game Over");
        cash = 500;
        askover();
    }
    if (cash > 1000000){
        FILE* fp=fopen("flag", "r");
        char buf[100];
        memset(buf, 0, 100);
        fread(buf, 1, 100, fp);
        printf("%s\n", buf);
        fclose(fp);
    }
}
```

---

## 💡 Goal

We need to make:

```
cash > 1000000
```

Initial value:

```
cash = 500
```

---

## ⚠️ Vulnerability

The betting system does not validate negative inputs.

```c
scanf("%d", &bet);
```

And the check is only:

```c
if (bet > cash)
```

👉 There is NO check for:

```
bet < 0
```

---

## 💣 Exploit Logic

When the player loses:

```c
cash = cash - bet;
```

If `bet` is negative:

```
cash = cash - (-X)
     = cash + X
```

👉 Losing the game actually **increases cash**

---

## 🔑 Exploit Strategy

1. Start the game
2. Enter a negative bet
3. Intentionally lose
4. Repeat or use a large negative value

---

## 🚀 Payload

Example:

```
-2000000000
```

---

## 🧮 Result

```
cash = 500 - (-2000000000)
     = 2000000500
```

This satisfies:

```
cash > 1000000
```

👉 Flag is printed immediately

---

## 🧪 Execution

```
nc pwnable.kr 9009
```

Then:

```
Y
1
-2000000000
```

(Proceed until loss or game logic applies)

---

## ⚙️ Execution Flow

```
user input (negative bet)
        ↓
program accepts input without validation
        ↓
loss occurs
        ↓
cash increases instead of decreasing
        ↓
cash exceeds 1,000,000
        ↓
flag is printed
```

---

## 📌 What I Learned

- `scanf("%d")` does not validate input ranges
- Negative values can break program logic
- Always validate both upper and lower bounds
- Game logic can be exploited without memory corruption

---

## 🧾 Summary

This challenge is not about winning Blackjack, but about exploiting flawed input validation.

By providing a negative bet value, we can reverse the intended logic and increase our cash, allowing us to reach the flag condition instantly.

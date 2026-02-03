# Lab 4 - Buffer-Overflow Attacks and Exploit Frameworks

## Buffer Vulnerabilites in C/C++

Buffer overflow vulnerabilities are among the oldest and most well-known security flaws in computer systems. 
Despite decades of research and mitigation techniques, they continue to appear in modern software. This lab focuses on understanding **why C and C++ programs are especially prone to buffer overflows**, and how poor programming practices can lead to exploitable vulnerabilities.

Historically, buffer overflows have been used in real-world attacks such as the Morris Worm (1988), and they are still relevant today, as discussed in classic papers like *Smashing the Stack for Fun and Profit* and modern security talks such as RSA Conference 2024.

## What is a Buffer Overflow?

A buffer overflow occurs when a program writes more data into a memory buffer than it was allocated to hold. In low-level languages such as C and C++, this extra data can overwrite adjacent memory locations, potentially corrupting variables, control data, or even program execution flow.

On stack-based memory layouts, overflowing a buffer can overwrite the function's **return address**, allowing an attacker to redirect execution to attacker-controlled code.

## Common Problems in C/C++ That Allow Buffer Overflows

One of the main reasons buffer overflows occur in C and C++ is the lack of automatic bounds checking. The languages allow direct memory access, and it is entirely the programmer’s responsibility to ensure that data written to a buffer does not exceed its allocated size. Writing past a buffer results in undefined behavior and may overwrite adjacent memory.

Additionally, C and C++ provide unsafe standard library functions such as `gets()`, `strcpy()`, and `sprintf()` that do not validate input length. When such functions are used with stack-allocated buffers, it becomes possible to overwrite control data like return addresses, which can lead to execution flow hijacking.

# Stacks!!!

**Stack Zero** reads user input from Standard input using unsafe `gets()` function and stores it in a fixed-size buffer. The program contains a local `buffer[64]` and a variable `changeme` initialized to `0`.
Because `gets()` does not perform bounds checking, suppyling more than 64 characters causes the input to overflow the buffer and overwrite adjacent stack memory. By entering a string longer than the buffer size the `changeme` variable is overwritten and its value changes from `0`, which triggers the success condition!

**The solution I used:**
```c
python3 -c 'print("A"*65)' | ./stack_zero
```
---
**Stack One**, user input is provided as a command-line argument and copied into a fixed-size stack buffer wtihout bounds checking. The program contains a `buffer[64]` and a variable `changeme`, which must be set to the value `0x496c5962` in order to succeed.

By suppyling an argument longer than 64 bytes, the buffer can be overflowed and the adjacent `changeme` variable overwritten. Since the system uses little-endian byte order, the value `0x496c5962` must be written as the byte sequence `bYlI`.

The `SECRET_PATTERN` that successfully solves Stack One is therefore `64 bytes of padding` + `bYlI`:

```c
./stack_one $(python -c 'print("A"*64 + "bYlI")')
```
This overwrites the `changeme` with the correct value and triggers the success message.

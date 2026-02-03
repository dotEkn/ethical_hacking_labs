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

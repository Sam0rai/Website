---
layout: striped-default
title: "EDR Evasion Using Indirect Memory Writing"
date: 2026-03-03
tags: [rea-team, evasion]
---

## Throwback to college
When I started my Bachelor’s Degree in Computer Science, one of our first courses was an introduction to C++, where we learned all about variables, conditions, loops, recursion, and of course – pointers. <br>
Later on we proceeded to learn more advanced topics, such as inheritance, polymorphism and… pointers to pointers.
While we were told that our variables and functions needed to be documented and have concise and revealing names to better understand their purpose, a friend of mine stuck to laconic phrasing, with naming conventions such as “bigNum” for a variable or “makePointer” for a function. His best idea for a name for a function that creates a pointer to a given pointer was: “make_pp”. 😉
<!--more-->

Unlike us college students back in the day, the Windows internals API has (for the most of it) concise names for its functions. When we see functions such as _“WriteFile”_ or _“ReadProcessMemory”_, we’ve got a pretty good idea of what they are supposed to do.
AV and EDR vendors follow the same logic; functions such as _“WriteProcessMemory”_, _“memcpy”_ and other direct buffer overwrites are well–hooked by antivirus and endpoint detection systems, and the first thing defenders monitor when malware tries to build or inject payloads.
But what if I told you don’t need direct writes at all? <br>
“Indirect Memory Writing” is a somewhat stealthy method, that abuses benign Windows APIs to write attacker–controlled bytes into memory without ever using a traditional memory writing primitive!
<br>

## What Is Indirect Memory Writing?

At its core: Instead of explicitly copying bytes into memory (e.g., via <i>“WriteProcessMemory”</i>), you use legitimate Windows APIs that already <u>write back status or completion values into pointers you control</u>.
These are APIs with “out” parameters – the very mechanisms the OS uses to report counts or results – only repurposed as write primitives.

For example: <br>
The function <i>“ReadFile”</i> accepts pointers where the OS stores the number of bytes written or read upon completion.
If you control that **“out”** pointer, you control where the OS writes the value – and thus can feed arbitrary bytes into a target buffer!<br>
From the defender’s perspective, these calls look like normal file or memory operations; no obvious memory-write behavior to trip heuristic scanners.
<img src="/assets-striped/posts\2026-03-03-EDR-Evasion-Using-Indirect-Memory-Writing/ReadFile-You-Keep-Using-That-Word.jpg" alt="You Keep Using That Word" class="center-image" style="max-width:100%; height:auto;">
<br><br>

## How Windows Memory APIs Enable the Trick
Let’s review the syntax for the “ReadProcessMemory” function:
<img src="/assets-striped/posts\2026-03-03-EDR-Evasion-Using-Indirect-Memory-Writing/ReadProcessMemory-Prototype.png" alt="ReadProcessMemory Prototype" class="center-image">
The official purpose (per Microsoft documentation) is simple: **Read data from another process’s memory** – providing:
- _hProcess_: handle to the target process
-	_lpBaseAddress_: address to read from
-	_lpBuffer_: where the read data goes
-	_nSize_: number of bytes to read
-	_lpNumberOfBytesRead_: pointer where the OS reports how many bytes were read <br>

This last one - the “out” parameter pointer - is where the magic happens!
Although documented as a read API, the OS writes the return count into the memory pointed to by this parameter. If that pointer points to an <u>arbitrary executable region you control</u>, that “status” write becomes an **injection primitive**. <br>
So instead of doing:
```C
WriteProcessMemory(targetProc, addr, buffer, size, &written);
```
You trigger:
```C
ReadProcessMemory(
  -1,
  dummyBuf,
  dummyBuf,
  payloadByte,
  targetMemory+offset
);
```
and let the OS write _payloadByte_ into _targetMemory+offset_ via the _lpNumberOfBytesRead_ pointer – byte by byte.
<img src="/assets-striped/posts\2026-03-03-EDR-Evasion-Using-Indirect-Memory-Writing/You-Were-Supposed-To-Read-From-Memory.jpg" alt="You Were Supposed To READ From Memory" class="center-image" style="max-width:100%; height:auto;">
<br>

## Why This Breaks the Defensive Model
Modern EDR and Antivirus solutions typically rely on: <br>
✔️ API hook monitoring <br>
✔️ Signature matching <br>
✔️ Behavioral heuristics  tied to known write primitives (e.g., _WriteProcessMemory_, _VirtualAlloc + memcpy_) <br><br>
But indirect memory writing never calls those directly. Instead, it hides in fully legitimate API usage. Security vendors haven’t usually looked for unusual write targets via “read” APIs, so this technique creates a blind spot: A write primitive that looks like normal Windows behavior.
<br>

## Real Proof-of-Concept: Indirect-Shellcode-Executor
The GitHub project [Indirect‑Shellcode‑Executor](https://github.com/mimorep/Indirect-Shellcode-Executor) on GitHub by researcher **Mimorep** shows a full offensive tool that operationalizes this trick in Rust.
It does three powerful things:
1.	**Remote Payload Execution** – Fetches shellcode from a C2 server (sometimes hidden inside benign files like images) without direct writes.
2.	**Terminal Injection** – Injects raw shellcode provided via CLI input.
3.	**File-Based Execution** – Loads payloads from local files and write them into executable memory indirectly.

By forcing the OS to write shellcode bytes via an “unhooked” path, it creates a write primitive beneath the radar of tools that only monitor known write APIs.
<br>

## In Conclusion
Indirect memory writing turns a 30-year-old API design choice (“out” pointers) into a weapon. It’s a reminder that every legitimate API behavior is also an attack surface, and that defensive models must evolve from names and signatures to semantics and context. <br>
This technique is not just clever – it’s dangerous, and it’s reshaping how we think about stealthy memory manipulation on Windows.

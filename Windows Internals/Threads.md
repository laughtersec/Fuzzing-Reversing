---
tags:
  - windows-internals
---
# Definition
A thread is an entity within a process that Windows schedules for execution. Without it, the program can't run.

A thread includes the following essential components:
- The contents of a set of CPU registers representing the state of the processor.
- There are two stacks for each thread - user and kernel.
- A private storage area called thread-local storage (TLS) for use by subsystems, run-time libraries, and DLLs.
- A unique identifier called a thread ID (part of an internal structure called client ID. Process IDs and thread IDs are generated out of the same namespace, so they never overlap).
In addition, threads sometimes have their own security context, or token, which is often used by 
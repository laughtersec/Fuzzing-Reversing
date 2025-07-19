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
In addition, threads sometimes have their own security context, or token, which is often used by multi-threaded server applications that impersonate the security context of the clients that they serve.

The volatile registers, stacks, and private storage area are called the thread's context. ==Because this information is different for each machine architecture that Windows runs on, this structure, by necessity, is architecture specific==

![[thread.png]]

Here's an interesting fact about threads - There's a good chance that the execution of the thread and the loading of libraries might overlap, causing runtime errors where the thread fails to

# Internals

## Various STructures

### Thread Environment Block
- Small memory range
- Provides storage for thread-specific information
	- Thread ID
	- Stack range
	- GetLastError
	- TLS: Thread local storage
- gs:[X] = [IA32_KERNEL_GS_BASE + X]


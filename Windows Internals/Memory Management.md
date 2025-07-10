---
tags:
  - windows-internals
---
# Where is it?
In ==`Ntoskrnl.exe`==. It is the largest component in the executive, hinting at its importance and complexity. It does NOT exist in the HAL.

# Components
- A set of executive system services for allocating, deallocating, and managing virtual memory, most of which are exposed through the Windows API or kernel-mode device driver interfaces.
- A translation-not-valid and access fault trap handler for resolving hardware-detected memory-management exceptions and making virtual pages resident on behalf of a process.
---
tags:
  - windows-internals
---
# Where is it?
In ==`Ntoskrnl.exe`==. It is the largest component in the executive, hinting at its importance and complexity. It does NOT exist in the HAL.

# Components
- A set of executive system services for allocating, deallocating, and managing virtual memory, most of which are exposed through the Windows API or kernel-mode device driver interfaces.
- A translation-not-valid and access fault trap handler for resolving hardware-detected memory-management exceptions and making virtual pages resident on behalf of a process.
- Six key top-level routines, each running in one of six different kernel-mode threads in the System process:
	- **The balance set manager (KeBalanceSetManager, priority 17)**: This calls an inner routine, the working set manager (MmWorkingSetManager), once per second as well as when free memory falls below a certain threshold. The working set manager drives the overall memory-management policies, such as working set trimming, aging, and modified page writing.
	- **The process/stack swapper (KeSwapProcessOrStack, priority 23)**: This performs both process and kernel thread
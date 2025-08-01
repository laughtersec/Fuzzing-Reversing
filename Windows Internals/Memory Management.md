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
	- **The process/stack swapper (KeSwapProcessOrStack, priority 23)**: This performs both process and kernel thread stack inswapping and outswapping. The balance set manager and the thread-scheduling code in the kernel awaken this thread when an inswap or outswap operation needs to take place.
	- **The modified page writer (MiModifiedPageWriter, pririoty 18)**: This writes dirty pages on the modified list back to the appropriate paging files. This thread is awakened when the size of the modified list needs to be reduced.
	- **The mapped page writer (MiMappedPageWriter, priority 18)**: This writes dirty pages in mapped files to disk or remote storage. It is awakened when the size of the modified list needs to be reduced or if pages for mapped files have been on the modified list for more than 5 minutes. This second modified page writer thread is necessary because it can generate page faults that result in requests for free pages.
	- **The segment dereference thread (MiDereferenceSegmentThread, priority 19)**: This is responsible for cache reduction as well as for page file growth and shrinkage. For example, if there is no virtual address space for paged pool growth, this thread trims the page cache so that the paged pool used to anchor it can be freed for reuse.
	- **The zero page thread (MiZeroPageThread, priority 0)**: This zeroes out pages on the free list so that a cache of zero pages is available to satisfy future demand-zero page faults. In some cases, memory zeroing is done by a faster function called `MiZeroInParallel`.

# Paging
Memory management is done in distinct chunks called pages. This is because the hardware memory management unit translates virtual to physical addresses at the granularity of a page. Hence, a page is the smallest unit of protection at the hardware level.

| Architecture | Small Page Size | Large Page Size | Small Pages per Large Page |
| ------------ | --------------- | --------------- | -------------------------- |
| x86 (PAE)    | 4 KB            | 2 MB            | 512                        |
| x64          | 4 KB            | 2 MB            | 512                        |
| ARM          | 4 KB            | 4 MB            | 1024                       |

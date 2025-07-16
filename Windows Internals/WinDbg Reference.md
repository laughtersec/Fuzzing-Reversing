---
tags:
  - windbg
---
# Documentation
WinDbg, despite its awful GUI, has this very amazing thing called the "Help" button, which historically has been present in any windows GUI based softwares.

![[ignored_help_page.png]]

![[memory_contents_windbg.png]]

I will never understand why so many of you never bother exploring.

## Enable Kernel Debugging

```powershell
bcdedit -debug on
```



## Setting a breakpoint on the main function

```d
bp $exentry
```


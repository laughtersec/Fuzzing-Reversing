---
tags:
  - windbg
---
# Documentation
WinDbg, despite its awful GUI, has this very amazing thing called the "Help" button, which historically has been present in any windows GUI based softwares.

![[ignored_help_page.png]]

![[memory_contents_windbg.png]]

I will never understand why so many of you never bother exploring.

## Kernel Debugging

```powershell title:"restart the system after running this command"
bcdedit -debug on
```

Kernel debugging should now work properly

![[kdebug_howto.png]]

![[kdebug_local.png]]

### Examining a Process using Kernel Debug

The desired process will have to be searched in the task manager and the ==hexadecimal of the PID== needs to be noted down to examine it in kernel debugging ( using base 10 PID values seems to fail).

```d
lkd> !process 3d00
Searching for Process with Cid == 3d00
PROCESS ffff9b07df2da0c0
    SessionId: none  Cid: 3d00    Peb: 0021e000  ParentCid: 51c8
    DirBase: 1c30473000  ObjectTable: ffffae03f0c86080  HandleCount:  52.
    Image: handles.exe
    VadRoot ffff9b07dbe27840 Vads 34 Clone 0 Private 132. Modified 0. Locked 0.
    DeviceMap ffffae03d93b9790
    Token                             ffffae03ec510770
    ElapsedTime                       00:08:42.139
    UserTime                          00:00:00.000
    KernelTime                        00:00:00.000
    QuotaPoolUsage[PagedPool]         29040
    QuotaPoolUsage[NonPagedPool]      4904
    Working Set Sizes (now,min,max)  (1141, 50, 345) (4564KB, 200KB, 1380KB)
    PeakWorkingSetSize                1115
    VirtualSize                       4145 Mb
    PeakVirtualSize                   4148 Mb
    PageFaultCount                    1194
    MemoryPriority                    BACKGROUND
    BasePriority                      8
    CommitCharge                      142
    Job                               ffff9b07dcb8a060
        
        THREAD ffff9b07df0f5080  Cid 3d00.65e0  Teb: 000000000021f000 Win32Thread: 0000000000000000 WAIT: (DelayExecution) UserMode Non-Alertable
            ffffffffffffffff  NotificationEvent
        Not impersonating
        DeviceMap                 ffffae03d93b9790
        Owning Process            ffff9b07df2da0c0       Image:         handles.exe
        Attached Process          N/A            Image:         N/A
        Wait Start TickCount      127693         Ticks: 33607 (0:00:08:45.109)
        Context Switch Count      8              IdealProcessor: 2             
        UserTime                  00:00:00.000
        KernelTime                00:00:00.000
        Win32 Start Address 0x00007ff6766a1430
        Stack Init ffff9885c5e2f530 Current ffff9885c5e2efe0
        Base ffff9885c5e30000 Limit ffff9885c5e29000 Call 0000000000000000
        Priority 9 BasePriority 8 PriorityDecrement 0 IoPriority 2 PagePriority 5
        Child-SP          RetAddr               Call Site
        ffff9885`c5e2f020 fffff804`8bf67be0     nt!KiSwapContext+0x76
        ffff9885`c5e2f160 fffff804`8bf26051     nt!KiSwapThread+0x6a0
        ffff9885`c5e2f230 fffff804`8be903ef     nt!KiCommitThreadWait+0x271
        ffff9885`c5e2f2d0 fffff804`8c461bce     nt!KeDelayExecutionThread+0x47f
        ffff9885`c5e2f370 fffff804`8c2b8d55     nt!NtDelayExecution+0x5e
        ffff9885`c5e2f3a0 00007ffb`71042454     nt!KiSystemServiceCopyEnd+0x25 (TrapFrame @ ffff9885`c5e2f3a0)
        00000000`0014fbd8 00000000`00000000     0x00007ffb`71042454
```

### Retrieve handles used by a process

```d title:"It is a very long list"
!handle 0 0x20 3d00
<...>
```

Perhaps I need to write a script to strategically dump information about interesting handles.
## Miscellaneous

```d title:"add a breakpoint to main"
bp $exentry
```


---
tags:
  - windows-internals
---
# Definition
It is a container for a set of resources used when executing the instance of the program and consists of the following:

| **Fundamental Component**        | **Purpose**                                                                                                                                                                                                                                                                                                           |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| A private virtual address space  | Set a virtual memory addresses that the process can use.                                                                                                                                                                                                                                                              |
| An executable program            | This defines initial code and data and is mapped into the process's virtual address space.                                                                                                                                                                                                                            |
| A list of open handles           | These map to various system resources such as semaphores, synchronization objects, and files that are accessible to all threads in the process                                                                                                                                                                        |
| A security context               | This is an access token that identifies the user, security groups, privileges, attributes, claims, capabilities, User Account Control (UAC) virtualization state, session, and limited user account state associated with the process, as well as the AppContainer identifier and its related sandboxing information. |
| A process ID                     | This is a unique identifier, which is internally part of an identifier called a client ID.                                                                                                                                                                                                                            |
| At least one thread of execution | Although an "empty" process is possible, it is (mostly) not useful.                                                                                                                                                                                                                                                   |

# Translation
Note: This is an analogy.

| **Gang Assets**                       | Purpose                                                                                                |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Land                                  | Some land (space) is required to carry out illegal activities in the landscape that is the world (RAM) |
| Manifesto                             | Some devil contract that they follow rigorously, around which all other assets work.                   |
| Blackmailed informants                | Have a handle on some person of some other entity that they use for their devious purposes.            |
| Tattoos for recognizing a gang member | Any communication will happen only between goons that are part of the same gang. Never snitch!         |
| A Name                                | Gangs have a name to uniquely identify them from one another (like how PIDs are unique)                |
| Goons                                 | Muscle that does the dirty work. They are the gang in action.                                          |

# Diagram
![[processes.svg]]

# Creating a Process
==The Windows API provides several functions for creating processes==. The simplest is `CreateProcess`, which attempts to create a process with the same access token as the creating process. If a different token is required, `CreateProcessAsUser` can be used, which accepts an extra argument (the first) - a handle to a token object that was already somehow obtained (for example, by calling the `LogonUser` function).

# Access Tokens
https://learn.microsoft.com/en-us/windows/win32/secauthz/access-tokens

An access token is an object that describes the *security context* of a process or [[Threads|thread]]. The information in a token includes the identity and privileges of the user account associated with the process or thread. When a user logs on, the system verifies the user's password by comparing it with information stored in a security database. If the password is *authenticated*, the system produces an access token. Every process executed on behalf of this user has a copy of this access token.

# Virtual Address Descriptor (VAD)
A fundamental component in the [[Memory Management|memory managment]] system of Windows. Primarily, it is responsible for managing and tracking the memory allocations within a process's virtual address space. VADs reside within the private memory space of each process and are managed by the Windows Kernel as part of the process's Virtual Memory Control Block (VMCB).

It represents a layer of abstraction between the physical memory (RAM) installed in a computer and the memory addresses that software applications use. When a process allocates memory (like by using VirtualAlloc), an entry is created in the VAD tree.

The structure of a VAD is a binary tree, with each node representing a block of virtual memory. These nodes contain crucial information about the memory block, including its starting address, size and state.

An example: A DLL is loaded into a process, a VAD node is created to record its memory region, access permissions, and the file it's mapped to.

By examining a process in windbg, the `VadRoot` can be found.

[The Windows Concept Journey — VAD Tree (Virtual Address Descriptor Tree) | by Shlomi Boutnaru, Ph.D. | Medium](https://medium.com/@boutnaru/the-windows-concept-journey-vad-tree-virtual-address-descriptor-tree-5cd9a1cc3f53)

# Internals
Each Windows process is represented by an executive process (`EPROCESS` structure). Besides containing many attributes relating to a process (such as `UniqueProcessId`), an `EPROCESS` contains and points to a number of other related data structures. For example, each process has one or more threads, each represented by an executive thread (`ETHREAD`) structure.

The `EPROCESS` and most of its related data structures exist in system address space. ==One exception is the Process Environment Block (PEB)==, which exists in the process (user) address space (because it contains information accessed by user-mode code). Additionally, some of the process data structures used in memory management, such as the working set list, are valid only within the context of the current process, because they are stored in process-specific system space.

## Various Structures
- For each process that is executing a Windows program, the Windows subsystem process (Csrss) maintains a parallel structure called the CSR_PROCESS.
- Windows subsystem (Win32k.sys) maintains a per-process data structure, `W32PROCESS`, which is created the first time a thread calls a Windows USER or GDI function that is implemented in kernel mode. This happens as soon as the User32.dll library is loaded (by calling functions like `CreateWindowEx`).
- The Graphics Device Interface (GDI) component infrastructure causes the DirectX Graphics Kernel (Dxgkrnl.sys) to initialize a structure of its own, `DXGPROCESS`.
Except for the idle process, every `EPROCESS` structure is encapsulated as a process object by the ==executive object manager==.

```d title:"Display type (dt) in nt!_eprocess of notepad.exe"
0:000> dt nt!_eprocess
ntdll!_EPROCESS
   +0x000 Pcb              : _KPROCESS
   +0x1c8 ProcessLock      : _EX_PUSH_LOCK
   +0x1d0 UniqueProcessId  : Ptr64 Void
   +0x1d8 ActiveProcessLinks : _LIST_ENTRY
   +0x1e8 RundownProtect   : _EX_RUNDOWN_REF
   +0x1f0 Flags2           : Uint4B
   +0x1f0 JobNotReallyActive : Pos 0, 1 Bit
   +0x1f0 AccountingFolded : Pos 1, 1 Bit
   +0x1f0 NewProcessReported : Pos 2, 1 Bit
   +0x1f0 ExitProcessReported : Pos 3, 1 Bit
   +0x1f0 ReportCommitChanges : Pos 4, 1 Bit
   +0x1f0 LastReportMemory : Pos 5, 1 Bit
   +0x1f0 ForceWakeCharge  : Pos 6, 1 Bit
   +0x1f0 CrossSessionCreate : Pos 7, 1 Bit
   +0x1f0 NeedsHandleRundown : Pos 8, 1 Bit
   +0x1f0 RefTraceEnabled  : Pos 9, 1 Bit
   +0x1f0 PicoCreated      : Pos 10, 1 Bit
   +0x1f0 EmptyJobEvaluated : Pos 11, 1 Bit
   +0x1f0 DefaultPagePriority : Pos 12, 3 Bits
   +0x1f0 PrimaryTokenFrozen : Pos 15, 1 Bit
   +0x1f0 ProcessVerifierTarget : Pos 16, 1 Bit
   +0x1f0 RestrictSetThreadContext : Pos 17, 1 Bit
   +0x1f0 AffinityPermanent : Pos 18, 1 Bit
   +0x1f0 AffinityUpdateEnable : Pos 19, 1 Bit
   +0x1f0 PropagateNode    : Pos 20, 1 Bit
   +0x1f0 ExplicitAffinity : Pos 21, 1 Bit
   +0x1f0 Flags2Available1 : Pos 22, 2 Bits
   +0x1f0 EnableReadVmLogging : Pos 24, 1 Bit
   +0x1f0 EnableWriteVmLogging : Pos 25, 1 Bit
   +0x1f0 FatalAccessTerminationRequested : Pos 26, 1 Bit
   +0x1f0 DisableSystemAllowedCpuSet : Pos 27, 1 Bit
   +0x1f0 Flags2Available2 : Pos 28, 3 Bits
   +0x1f0 InPrivate        : Pos 31, 1 Bit
   +0x1f4 Flags            : Uint4B
   +0x1f4 CreateReported   : Pos 0, 1 Bit
   +0x1f4 NoDebugInherit   : Pos 1, 1 Bit
   +0x1f4 ProcessExiting   : Pos 2, 1 Bit
   +0x1f4 ProcessDelete    : Pos 3, 1 Bit
   +0x1f4 ManageExecutableMemoryWrites : Pos 4, 1 Bit
   +0x1f4 VmDeleted        : Pos 5, 1 Bit
   +0x1f4 OutswapEnabled   : Pos 6, 1 Bit
   +0x1f4 Outswapped       : Pos 7, 1 Bit
   +0x1f4 FailFastOnCommitFail : Pos 8, 1 Bit
   +0x1f4 Wow64VaSpace4Gb  : Pos 9, 1 Bit
   +0x1f4 AddressSpaceInitialized : Pos 10, 2 Bits
   +0x1f4 SetTimerResolution : Pos 12, 1 Bit
   +0x1f4 BreakOnTermination : Pos 13, 1 Bit
   +0x1f4 DeprioritizeViews : Pos 14, 1 Bit
   +0x1f4 WriteWatch       : Pos 15, 1 Bit
   +0x1f4 ProcessInSession : Pos 16, 1 Bit
   +0x1f4 OverrideAddressSpace : Pos 17, 1 Bit
   +0x1f4 HasAddressSpace  : Pos 18, 1 Bit
   +0x1f4 LaunchPrefetched : Pos 19, 1 Bit
   +0x1f4 Reserved         : Pos 20, 1 Bit
   +0x1f4 VmTopDown        : Pos 21, 1 Bit
   +0x1f4 ImageNotifyDone  : Pos 22, 1 Bit
   +0x1f4 PdeUpdateNeeded  : Pos 23, 1 Bit
   +0x1f4 VdmAllowed       : Pos 24, 1 Bit
   +0x1f4 ProcessRundown   : Pos 25, 1 Bit
   +0x1f4 ProcessInserted  : Pos 26, 1 Bit
   +0x1f4 DefaultIoPriority : Pos 27, 3 Bits
   +0x1f4 ProcessSelfDelete : Pos 30, 1 Bit
   +0x1f4 SetTimerResolutionLink : Pos 31, 1 Bit
   +0x1f8 CreateTime       : _LARGE_INTEGER
   +0x200 ProcessQuotaUsage : [2] Uint8B
   +0x210 ProcessQuotaPeak : [2] Uint8B
   +0x220 PeakVirtualSize  : Uint8B
   +0x228 VirtualSize      : Uint8B
   +0x230 SessionProcessLinks : _LIST_ENTRY
   +0x240 ExceptionPortData : Ptr64 Void
   +0x240 ExceptionPortValue : Uint8B
   +0x240 ExceptionPortState : Pos 0, 3 Bits
   +0x248 Token            : _EX_FAST_REF
   +0x250 MmReserved       : Uint8B
   +0x258 AddressCreationLock : _EX_PUSH_LOCK
   +0x260 PageTableCommitmentLock : _EX_PUSH_LOCK
   +0x268 RotateInProgress : Ptr64 _ETHREAD
   +0x270 ForkInProgress   : Ptr64 _ETHREAD
   +0x278 CommitChargeJob  : Ptr64 _EJOB
   +0x280 CloneRoot        : _RTL_AVL_TREE
   +0x288 NumberOfPrivatePages : Uint8B
   +0x290 NumberOfLockedPages : Uint8B
   +0x298 Win32Process     : Ptr64 Void
   +0x2a0 Job              : Ptr64 _EJOB
   +0x2a8 SectionObject    : Ptr64 Void
   +0x2b0 SectionBaseAddress : Ptr64 Void
   +0x2b8 Cookie           : Uint4B
   +0x2c0 WorkingSetWatch  : Ptr64 _PAGEFAULT_HISTORY
   +0x2c8 Win32WindowStation : Ptr64 Void
   +0x2d0 InheritedFromUniqueProcessId : Ptr64 Void
   +0x2d8 OwnerProcessId   : Uint8B
   +0x2e0 Peb              : Ptr64 _PEB
   +0x2e8 Session          : Ptr64 _PSP_SESSION_SPACE
   +0x2f0 Spare1           : Ptr64 Void
   +0x2f8 QuotaBlock       : Ptr64 _EPROCESS_QUOTA_BLOCK
   +0x300 ObjectTable      : Ptr64 _HANDLE_TABLE
   +0x308 DebugPort        : Ptr64 Void
   +0x310 WoW64Process     : Ptr64 _EWOW64PROCESS
   +0x318 DeviceMap        : _EX_FAST_REF
   +0x320 EtwDataSource    : Ptr64 Void
   +0x328 PageDirectoryPte : Uint8B
   +0x330 ImageFilePointer : Ptr64 _FILE_OBJECT
   +0x338 ImageFileName    : [15] UChar
   +0x347 PriorityClass    : UChar
   +0x348 SecurityPort     : Ptr64 Void
   +0x350 SeAuditProcessCreationInfo : _SE_AUDIT_PROCESS_CREATION_INFO
   +0x358 JobLinks         : _LIST_ENTRY
   +0x368 HighestUserAddress : Ptr64 Void
   +0x370 ThreadListHead   : _LIST_ENTRY
   +0x380 ActiveThreads    : Uint4B
   +0x384 ImagePathHash    : Uint4B
   +0x388 DefaultHardErrorProcessing : Uint4B
   +0x38c LastThreadExitStatus : Int4B
   +0x390 PrefetchTrace    : _EX_FAST_REF
   +0x398 LockedPagesList  : Ptr64 Void
   +0x3a0 ReadOperationCount : _LARGE_INTEGER
   +0x3a8 WriteOperationCount : _LARGE_INTEGER
   +0x3b0 OtherOperationCount : _LARGE_INTEGER
   +0x3b8 ReadTransferCount : _LARGE_INTEGER
   +0x3c0 WriteTransferCount : _LARGE_INTEGER
   +0x3c8 OtherTransferCount : _LARGE_INTEGER
   +0x3d0 CommitChargeLimit : Uint8B
   +0x3d8 CommitCharge     : Uint8B
   +0x3e0 CommitChargePeak : Uint8B
   +0x400 Vm               : _MMSUPPORT_FULL
   +0x540 MmProcessLinks   : _LIST_ENTRY
   +0x550 ModifiedPageCount : Uint4B
   +0x554 ExitStatus       : Int4B
   +0x558 VadRoot          : _RTL_AVL_TREE
   +0x560 VadHint          : Ptr64 Void
   +0x568 VadCount         : Uint8B
   +0x570 VadPhysicalPages : Uint8B
   +0x578 VadPhysicalPagesLimit : Uint8B
   +0x580 AlpcContext      : _ALPC_PROCESS_CONTEXT
   +0x5a0 TimerResolutionLink : _LIST_ENTRY
   +0x5b0 TimerResolutionStackRecord : Ptr64 _PO_DIAG_STACK_RECORD
   +0x5b8 RequestedTimerResolution : Uint4B
   +0x5bc SmallestTimerResolution : Uint4B
   +0x5c0 ExitTime         : _LARGE_INTEGER
   +0x5c8 InvertedFunctionTable : Ptr64 _INVERTED_FUNCTION_TABLE_USER_MODE
   +0x5d0 InvertedFunctionTableLock : _EX_PUSH_LOCK
   +0x5d8 ActiveThreadsHighWatermark : Uint4B
   +0x5dc LargePrivateVadCount : Uint4B
   +0x5e0 ThreadListLock   : _EX_PUSH_LOCK
   +0x5e8 WnfContext       : Ptr64 Void
   +0x5f0 ServerSilo       : Ptr64 _EJOB
   +0x5f8 SignatureLevel   : UChar
   +0x5f9 SectionSignatureLevel : UChar
   +0x5fa Protection       : _PS_PROTECTION
   +0x5fb HangCount        : Pos 0, 3 Bits
   +0x5fb GhostCount       : Pos 3, 3 Bits
   +0x5fb PrefilterException : Pos 6, 1 Bit
   +0x5fc Flags3           : Uint4B
   +0x5fc Minimal          : Pos 0, 1 Bit
   +0x5fc ReplacingPageRoot : Pos 1, 1 Bit
   +0x5fc Crashed          : Pos 2, 1 Bit
   +0x5fc JobVadsAreTracked : Pos 3, 1 Bit
   +0x5fc VadTrackingDisabled : Pos 4, 1 Bit
   +0x5fc AuxiliaryProcess : Pos 5, 1 Bit
   +0x5fc SubsystemProcess : Pos 6, 1 Bit
   +0x5fc IndirectCpuSets  : Pos 7, 1 Bit
   +0x5fc RelinquishedCommit : Pos 8, 1 Bit
   +0x5fc HighGraphicsPriority : Pos 9, 1 Bit
   +0x5fc CommitFailLogged : Pos 10, 1 Bit
   +0x5fc ReserveFailLogged : Pos 11, 1 Bit
   +0x5fc SystemProcess    : Pos 12, 1 Bit
   +0x5fc AllImagesAtBasePristineBase : Pos 13, 1 Bit
   +0x5fc AddressPolicyFrozen : Pos 14, 1 Bit
   +0x5fc ProcessFirstResume : Pos 15, 1 Bit
   +0x5fc ForegroundExternal : Pos 16, 1 Bit
   +0x5fc ForegroundSystem : Pos 17, 1 Bit
   +0x5fc HighMemoryPriority : Pos 18, 1 Bit
   +0x5fc EnableProcessSuspendResumeLogging : Pos 19, 1 Bit
   +0x5fc EnableThreadSuspendResumeLogging : Pos 20, 1 Bit
   +0x5fc SecurityDomainChanged : Pos 21, 1 Bit
   +0x5fc SecurityFreezeComplete : Pos 22, 1 Bit
   +0x5fc VmProcessorHost  : Pos 23, 1 Bit
   +0x5fc VmProcessorHostTransition : Pos 24, 1 Bit
   +0x5fc AltSyscall       : Pos 25, 1 Bit
   +0x5fc TimerResolutionIgnore : Pos 26, 1 Bit
   +0x5fc DisallowUserTerminate : Pos 27, 1 Bit
   +0x5fc EnableProcessRemoteExecProtectVmLogging : Pos 28, 1 Bit
   +0x5fc EnableProcessLocalExecProtectVmLogging : Pos 29, 1 Bit
   +0x5fc MemoryCompressionProcess : Pos 30, 1 Bit
   +0x5fc EnableProcessImpersonationLogging : Pos 31, 1 Bit
   +0x600 DeviceAsid       : Int4B
   +0x608 SvmData          : Ptr64 Void
   +0x610 SvmProcessLock   : _EX_PUSH_LOCK
   +0x618 SvmLock          : Uint8B
   +0x620 SvmProcessDeviceListHead : _LIST_ENTRY
   +0x630 LastFreezeInterruptTime : Uint8B
   +0x638 DiskCounters     : Ptr64 _PROCESS_DISK_COUNTERS
   +0x640 PicoContext      : Ptr64 Void
   +0x648 EnclaveTable     : Ptr64 Void
   +0x650 EnclaveNumber    : Uint8B
   +0x658 EnclaveLock      : _EX_PUSH_LOCK
   +0x660 HighPriorityFaultsAllowed : Uint4B
   +0x668 EnergyContext    : Ptr64 _PO_PROCESS_ENERGY_CONTEXT
   +0x670 VmContext        : Ptr64 Void
   +0x678 SequenceNumber   : Uint8B
   +0x680 CreateInterruptTime : Uint8B
   +0x688 CreateUnbiasedInterruptTime : Uint8B
   +0x690 TotalUnbiasedFrozenTime : Uint8B
   +0x698 LastAppStateUpdateTime : Uint8B
   +0x6a0 LastAppStateUptime : Pos 0, 61 Bits
   +0x6a0 LastAppState     : Pos 61, 3 Bits
   +0x6a8 SharedCommitCharge : Uint8B
   +0x6b0 SharedCommitLock : _EX_PUSH_LOCK
   +0x6b8 SharedCommitLinks : _LIST_ENTRY
   +0x6c8 AllowedCpuSets   : Uint8B
   +0x6d0 DefaultCpuSets   : Uint8B
   +0x6c8 AllowedCpuSetsIndirect : Ptr64 Uint8B
   +0x6d0 DefaultCpuSetsIndirect : Ptr64 Uint8B
   +0x6d8 DiskIoAttribution : Ptr64 Void
   +0x6e0 DxgProcess       : Ptr64 Void
   +0x6e8 Win32KFilterSet  : Uint4B
   +0x6ec Machine          : Uint2B
   +0x6ee MmSlabIdentity   : UChar
   +0x6ef Spare0           : UChar
   +0x6f0 ProcessTimerDelay : _PS_INTERLOCKED_TIMER_DELAY_VALUES
   +0x6f8 KTimerSets       : Uint4B
   +0x6fc KTimer2Sets      : Uint4B
   +0x700 ThreadTimerSets  : Uint4B
   +0x708 VirtualTimerListLock : Uint8B
   +0x710 VirtualTimerListHead : _LIST_ENTRY
   +0x720 WakeChannel      : _WNF_STATE_NAME
   +0x720 WakeInfo         : _PS_PROCESS_WAKE_INFORMATION
   +0x750 MitigationFlags  : Uint4B
   +0x750 MitigationFlagsValues : <unnamed-tag>
   +0x754 MitigationFlags2 : Uint4B
   +0x754 MitigationFlags2Values : <unnamed-tag>
   +0x758 PartitionObject  : Ptr64 Void
   +0x760 SecurityDomain   : Uint8B
   +0x768 ParentSecurityDomain : Uint8B
   +0x770 CoverageSamplerContext : Ptr64 Void
   +0x778 MmHotPatchContext : Ptr64 Void
   +0x780 DynamicEHContinuationTargetsTree : _RTL_AVL_TREE
   +0x788 DynamicEHContinuationTargetsLock : _EX_PUSH_LOCK
   +0x790 DynamicEnforcedCetCompatibleRanges : _PS_DYNAMIC_ENFORCED_ADDRESS_RANGES
   +0x7a0 DisabledComponentFlags : Uint4B
   +0x7a4 PageCombineSequence : Int4B
   +0x7a8 EnableOptionalXStateFeaturesLock : _EX_PUSH_LOCK
   +0x7b0 PathRedirectionHashes : Ptr64 Uint4B
   +0x7b8 SyscallProviderReserved : [4] Ptr64 Void
   +0x7d8 MitigationFlags3 : Uint4B
   +0x7d8 MitigationFlags3Values : <unnamed-tag>
   +0x7dc Flags4           : Uint4B
   +0x7dc ThreadWasActive  : Pos 0, 1 Bit
   +0x7dc MinimalTerminate : Pos 1, 1 Bit
   +0x7dc ImageExpansionDisable : Pos 2, 1 Bit
   +0x7dc SessionFirstProcess : Pos 3, 1 Bit
   +0x7e0 SyscallUsage     : Uint4B
   +0x7e0 SyscallUsageValues : <unnamed-tag>
   +0x7e4 SupervisorDeviceAsid : Int4B
   +0x7e8 SupervisorSvmData : Ptr64 Void
   +0x7f0 NetworkCounters  : Ptr64 _PROCESS_NETWORK_COUNTERS
   +0x7f8 Execution        : _PROCESS_EXECUTION
   +0x800 ThreadIndexTable : Ptr64 Void
   +0x808 FreezeWorkLinks  : _LIST_ENTRY
```

```d title:"Display type (dt) in PCB of notepad.exe"
0:000> dt nt!_kprocess
ntdll!_KPROCESS
   +0x000 Header           : _DISPATCHER_HEADER
   +0x018 ProfileListHead  : _LIST_ENTRY
   +0x028 DirectoryTableBase : Uint8B
   +0x030 ThreadListHead   : _LIST_ENTRY
   +0x040 ProcessLock      : Uint4B
   +0x044 ProcessTimerDelay : Uint4B
   +0x048 DeepFreezeStartTime : Uint8B
   +0x050 Affinity         : Ptr64 _KAFFINITY_EX
   +0x058 AutoBoostState   : _KAB_UM_PROCESS_CONTEXT
   +0x068 ReadyListHead    : _LIST_ENTRY
   +0x078 SwapListEntry    : _SINGLE_LIST_ENTRY
   +0x080 ActiveProcessors : Ptr64 _KAFFINITY_EX
   +0x088 AutoAlignment    : Pos 0, 1 Bit
   +0x088 DisableBoost     : Pos 1, 1 Bit
   +0x088 DisableQuantum   : Pos 2, 1 Bit
   +0x088 DeepFreeze       : Pos 3, 1 Bit
   +0x088 TimerVirtualization : Pos 4, 1 Bit
   +0x088 CheckStackExtents : Pos 5, 1 Bit
   +0x088 CacheIsolationEnabled : Pos 6, 1 Bit
   +0x088 PpmPolicy        : Pos 7, 4 Bits
   +0x088 VaSpaceDeleted   : Pos 11, 1 Bit
   +0x088 MultiGroup       : Pos 12, 1 Bit
   +0x088 ForegroundProcess : Pos 13, 1 Bit
   +0x088 ReservedFlags    : Pos 14, 18 Bits
   +0x088 ProcessFlags     : Int4B
   +0x08c Spare0c          : Uint4B
   +0x090 BasePriority     : Char
   +0x091 QuantumReset     : Char
   +0x092 Visited          : Char
   +0x093 Flags            : _KEXECUTE_OPTIONS
   +0x098 ActiveGroupsMask : _KGROUP_MASK
   +0x0a8 ActiveGroupPadding : [2] Uint8B
   +0x0b8 IdealProcessorAssignmentBlock : Ptr64 _KI_IDEAL_PROCESSOR_ASSIGNMENT_BLOCK
   +0x0c0 Padding          : [6] Uint8B
   +0x0f0 Padding2         : Uint4B
   +0x0f4 SchedulerAssistYieldBoostCount : Uint4B
   +0x0f8 SchedulerAssistYieldBoostAllowedTime : Int8B
   +0x100 Spare0d          : Uint4B
   +0x104 IdealGlobalNode  : Uint2B
   +0x106 Spare1           : Uint2B
   +0x108 StackCount       : _KSTACK_COUNT
   +0x110 ProcessListEntry : _LIST_ENTRY
   +0x120 CycleTime        : Uint8B
   +0x128 ContextSwitches  : Uint8B
   +0x130 SchedulingGroup  : Ptr64 _KSCHEDULING_GROUP
   +0x138 KernelTime       : Uint8B
   +0x140 UserTime         : Uint8B
   +0x148 ReadyTime        : Uint8B
   +0x150 FreezeCount      : Uint4B
   +0x154 Spare4           : Uint4B
   +0x158 UserDirectoryTableBase : Uint8B
   +0x160 AddressPolicy    : UChar
   +0x161 Spare2           : [7] UChar
   +0x168 InstrumentationCallback : Ptr64 Void
   +0x170 SecureState      : <unnamed-tag>
   +0x178 KernelWaitTime   : Uint8B
   +0x180 UserWaitTime     : Uint8B
   +0x188 LastRebalanceQpc : Uint8B
   +0x190 PerProcessorCycleTimes : Ptr64 Void
   +0x198 ExtendedFeatureDisableMask : Uint8B
   +0x1a0 PrimaryGroup     : Uint2B
   +0x1a2 Spare3           : [3] Uint2B
   +0x1a8 UserCetLogging   : Ptr64 Void
   +0x1b0 CpuPartitionList : _LIST_ENTRY
   +0x1c0 AvailableCpuState : Ptr64 _KPROCESS_AVAILABLE_CPU_STATE
```

### PROCESS CONTROL BLOCK
- It is a structure of type `KPROCESS`, for kernel process. 
- Although routines in the executive store information in the `EPROCESS`, the dispatcher, scheduler, and interrupt/time accounting code - being part of the operating system kernel - use the `KPROCESS` instead.

### Process Environment Block
- It exists in the process (user) address space (because it contains information accessed by user-mode code).
- Some of the process data structures used in memory management, such as the working set list, are valid only within the context of the current process, because they are stored in process-specific system space.
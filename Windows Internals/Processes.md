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

# Internals
Each Windows process is represented by an executive process (`EPROCESS` structure). Besides containing many attributes relating to a process (such as `UniqueProcessId`), an `EPROCESS` contains and points to a number of other related data structures. For example, each process has one or more threads, each represented by an executive thread (`ETHREAD`) structure.

The `EPROCESS` and most of its related data structures exist in system address space. ==One exception is the Process Environment Block (PEB)==, which exists in the process (user) address space (because it contains information accessed by user-mode code). Additionally, some of the process data structures used in memory management, such as the working set list, are valid only within the context of the current process, because they are stored in process-specific system space.

## Various Structures

- For each process that is executing a Windows program, the Windows subsystem process (Csrss) maintains a parallel structure called the CSR_PROCESS.
- 
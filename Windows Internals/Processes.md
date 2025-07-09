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
[Access Tokens - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-tokens)


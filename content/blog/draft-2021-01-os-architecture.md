+++
title = "Operating Systems Architecture"
slug = "2021/os-architecture"
#description = "a short description"

date = 2099-01-01
#updated = 2099-01-01

draft = true

[taxonomies]
tags = ["Operating Systems"]
+++


# Introduction
In this article, I will illustrate some operating system concepts with examples from Linux / Unix.

These concepts are tied to the architecture, so they still apply with other operating systems (for example, see [Windows user mode and kernel mode](https://docs.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)).


# Operating System Layers
Operating systems are organized in abstraction layers, bottom layer closer to the hardware, upper layers closer to user applications.

![Kernel Layout](/assets/2021/Operating_system_placement.svg)

<i>Source: <a href=https://en.wikipedia.org/wiki/Operating_system>https://en.wikipedia.org/wiki/Operating_system</a></i>

The kernel is responsible for:
* Interaction with hardware devices (drivers)
* Multitasking (process scheduling, permissions)
* Memory management (paging, permissions)
* Abstraction layers (Virtual File System)
* Networking
* CPU mode (rings)


# System Calls
User processes interact with the kernel using **system calls** or **syscalls**.

![System Calls](/assets/2021/os-architecture_syscall.svg)


Syscalls can be used to
* interact with files (open, read, write)
* interact with processes (create, kill)
* allocate / deallocate memory

Syscalls are either called directly by the user process, or called via a glibc wrapper, that also provide some POSIX features.

`strace` can be used to trace syscalls called by a program at runtime.


# Security Rings
Kernel and user space run into different CPU modes, usually represented as concentric rings.

![Rings](/assets/2021/os-architecture_Priv_rings.svg)

<i>Source: <a href=https://en.wikipedia.org/wiki/Protection_ring>https://en.wikipedia.org/wiki/Protection_ring</a></i>

In practice, operating systems only use ring 0 for the kernel, and ring 3 for user applications.

The inner rings have access to the outer rings, so the kernel has access to all applications.

Inner rings are more sensitive to security vulnerabilities, because the impact of a vulnerability could compromise the whole machine.


# User vs Root
On Unix, the user with UID 0 is known as the superuser, or `root`

`root` processes don't run in ring 0, they still run in ring 3, like any other user.

However, `root` can load/unload kernel modules, and those modules can run kernel code in ring 0.


# Sources & References
1. [Wikipedia - GNU C Library / glibc](https://en.wikipedia.org/wiki/GNU_C_Library)
1. [Wikipedia - Operating System](https://en.wikipedia.org/wiki/Operating_system)
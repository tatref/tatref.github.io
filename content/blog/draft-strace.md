+++
title = "System Calls & strace"
slug = "strace"
#description = "a short description"

date = 2099-01-01
#updated = 2099-01-01

draft = true

[taxonomies]
tags = ["linux", "strace"]
#authors = ["JB", "Alfred", "Nobel"]
+++

# Introduction
Programs interact with the operating system via system calls or syscalls.

TODO: diagram https://www.linuxbnb.net/wp-content/uploads/2018/06/system-call-overview-1-768x580.png

<i>Source: </i>

Syscalls are used to interact with:
* files: open, read, write, list
* network: open sockets, read, write
* memory: allocation, deallocation
* processes: creation, termination


# Tracing System Calls with `strace`
Tracing syscalls is done with the `strace` command

Useful options include
* `-p <pid>`: specify PID to trace
* `-f`: also trace children
* `-e <syscall list>`: specify which syscall you want to trace
* `-tt`: add timestamps
* `-T`: add syscall duration

Open a first terminal, and get its PID
```shell-session
$ echo $$
9431
```

Open a second terminal, and trace the `clone`, `execve`, and `exit_group` syscall
```shell-session
$ strace -f -e clone,execve,exit_group -p 9431
strace: Process 9431 attached
```

Run a command in the first terminal, and check the output of `strace`
```shell-session
$ strace -f -e clone,execve,exit_group -p 9431
strace: Process 9431 attached
clone(child_stack=NULL, flags=CLONE_CHILD_CLEARTID|CLONE_CHILD_SETTID|SIGCHLD, child_tidptr=0x7f88f2f7ca10) = 10006
strace: Process 10006 attached
[pid 10006] execve("/usr/bin/ls", ["ls", "--color=auto"], 0x55abc5fd5280 /* 39 vars */) = 0
[pid 10006] exit_group(0)               = ?
[pid 10006] +++ exited with 0 +++
^C
strace: Process 9431 detached
```

1. `clone()` is used to clone the current (shell) process. The return value is the PID of the new process
1. `execve()` will load the executable specified as first argument, and create a new stack/heap
1. After `ls` has done its work, it exits with `exit_group`


# Conclusion
Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.

# Sources & References
1. [Wikipedia - System call](https://en.wikipedia.org/wiki/System_call)
1. man strace
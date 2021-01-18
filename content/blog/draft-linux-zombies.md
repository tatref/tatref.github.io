+++
title = "Zombie processes"
slug = "2020/linux-zombie-processes"
#description = "What is a zombie process?"

date = 2099-01-01
#updated = 2020-10-10

draft = true

[taxonomies]
tags = ["system", "zombies", "linux"]
#authors = ["JB", "Alfred", "Nobel"]
+++


# What You'll Learn
1. How to create a zombie
1. How to get rid of it
1. Memory considerations


# Creating a zombie

Lets start with a script to create a zombie

```py3
#!/usr/bin/env python3

import os
import sys
import time


try:
    pid = os.fork()
except Exception as e:
    print('Fork failed')
    sys.exit(1)

if pid == 0:
    # child process
    print('Hello from zombie child ' + str(os.getpid()))
    sys.exit(0)

# parent process
time.sleep(0.1)

print('Press enter to wait()')
input()
os.waitpid(pid, 0)

print('Zombie child reaped')

print('Press enter to exit')
input()
```

Copy the code to a file `zombie1.py` and run it in one terminal. Press ENTER or CTRL-C to kill the zombie, don't do it yet!

```shell-session
$ python3 zombie.py
Hello from zombie child 7414
Press enter to wait()
```

# Inspecting

Before pressing ENTER, we can list the zombies in another terminal
```shell-session
$ ps -ely | grep ^Z
S   UID   PID  PPID  C PRI  NI   RSS    SZ WCHAN  TTY          TIME CMD
Z  1000  7416  7415  0  80   0     0     0 -      pts/2    00:00:00 python3 <defunct>
```

A few observations:
* Program is in state `Z`, and labeled `<defunct>`
* Program doesn't use any memory (`RSS` = 0)

We can also check `/proc/<pid>/smaps` to validate than the process doesn't have access to any memory
```shell-session
$ cat /proc/7416/smaps
$
```

The zombie doesn't have any file descriptor (not even `stdin`, `stdout`, or `stderr`)
```shell-session
$ ls -al /proc/7416/fd/
total 0
dr-x------ 2 root   root   0 Dec 18 22:42 .
dr-xr-xr-x 9 tatref tatref 0 Dec 18 22:14 ..
```

# How it works

Let's go into the script step by step.

First, we try to fork the current process
```py3
try:
    pid = os.fork()
except Exception as e:
    print('Fork failed')
    sys.exit(1)
```

If the `fork()` succeeded, we now have 2 processes.

When a process is created (a task in Linux parlance), an entry is inserted in the tasks list. Each entry contains a [list of open files](https://github.com/torvalds/linux/blob/71c061d2443814de15e177489d5cc00a4a253ef3/include/linux/sched.h#L973), the [state of the process](https://github.com/torvalds/linux/blob/71c061d2443814de15e177489d5cc00a4a253ef3/include/linux/sched.h#L658), the [exit status](https://github.com/torvalds/linux/blob/71c061d2443814de15e177489d5cc00a4a253ef3/include/linux/sched.h#L781])...

TODO: add linux PCB
http://sop.upv.es/gii-dso/en/t3-procesos-en-linux/gen-t3-procesos-en-linux.html
https://idea.popcount.org/2012-12-11-linux-process-states/


We use the returned `pid` value to check if we are the parent or the child.
The child just prints it's own pid, and exits
```py3
if pid == 0:
    # child
    print('Hello from zombie child ' + str(os.getpid()))
    sys.exit(0)

```

The parent will block until we press enter. It will then wait for the child process to terminate using `waitpid()`
```py3
print('Press enter to wait()')
input()
os.waitpid(pid, 0)
```

Here is what the lifecycle looks like:

![linux process lifcycle](/assets/2021/linux-zombies-process_lifecycle.svg)

There is no transition from `running` to `deleted`, a process has to become a zombie first.

Also note that there is no `dead` or `deleted` state: when the parent calls `wait()`, the child just ceases to exist if it was in the `zombie` state.

At that moment, the kernel passes the exit status of the child to the parent, and delete the process from the 


# SIGCHLD

```py3
import signal

def handler(signum, frame):
    print('Got signal', signum)

signal.signal(signal.SIGCHLD, handler)
```

When the child exits, the parent gets notified with `SIGCHLD` (17)
```shell-session
$ ./zombies.py 
Hello from child 8537
**Got signal 17**
Press enter to wait()
```


# Conclusion
- zombie is an expected state of a Linux program
- too many zombies could mean the parent process is having an issue

# Sources & References
1. Processes, forks, signals... - [BogoToBogo](https://www.bogotobogo.com/Linux/linux_process_and_signals.php)
1. [Redhat - Understanding Linux Process States](https://access.redhat.com/sites/default/files/attachments/processstates_20120831.pdf)
1. https://www.usna.edu/Users/cs/bilzor/ic411/calendar.php?type=class&event=6
1. https://man7.org/linux/man-pages/man2/waitid.2.html
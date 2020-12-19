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

# Intro
## Creating a zombie

Lets start with a little [script](/assets/2020/zombie.py) to generate a zombie

Download and run the script in one terminal. Press ENTER or CTRL-C will kill the zombie

```shell-session
$ curl -LO https://tatref.github.io/assets/2020/zombie.py
$ python3 zombie.py
Hello from zombie child 7414
Press enter to wait()

Zombie child reaped

Press enter to exit
```

## Inspecting

Before pressing ENTER, we can look for zombies
```shell-session
$ ps -ely | grep ^Z
S   UID   PID  PPID  C PRI  NI   RSS    SZ WCHAN  TTY          TIME CMD
Z  1000  7416  7415  0  80   0     0     0 -      pts/2    00:00:00 python3 <defunct>
```

* Program is in state `Z`, and labeled `<defunct>`
* Program doesn't use memory (`RSS` = 0)

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

We try to fork the current process
```py3
try:
    pid = os.fork()
except Exception as e:
    print('Fork failed')
    sys.exit(1)
```

We use the returned `pid` value to check if we are the parent or the child.

The child just prints it's own pid, and exits
```py3
if pid == 0:
    # child
    print('Hello from zombie child ' + str(os.getpid()))
    sys.exit(0)

```

The parent will block until we press enter. It will then `wait()` for the child process
```py3
print('Press enter to wait()')
input()
os.waitpid(pid, 0)
```

The 2 programs are scheduled by the OS task scheduler, so we can make no assumption on their states until the parent retuns from `waitpid()`.

These 2 states are possible:
1. child already exited when we hit `waitpid()`
1. child child is still running when we hit `waitpid()`

However, after `waitpid()`, the child transitions from zombie to dead

# Program lifecycle
https://access.redhat.com/sites/default/files/attachments/processstates_20120831.pdf

https://www.bogotobogo.com/Linux/linux_process_and_signals.php

SIGCHLD

We can see intermitent zombie


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
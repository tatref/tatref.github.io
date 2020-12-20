+++
title = "POSIX Conventions"
slug = "posix-conventions"
#description = "a short description"

date = 2020-12-20
#updated = 2099-01-01

#draft = true

[taxonomies]
tags = ["linux", "shell"]
+++


# Introduction
Nearly all conventions on Linux originate from Unix or POSIX


# The Shell Prompt
The prompt is what is displayed when you start a terminal, the default looks like
```shell-session
user@server:~$
```

This is useful to know which server we're connected to

I might abbreviate this to just `$` (simple user), `#` (root), or `>` (non-shell prompt)

For example
```shell-session
$ sudo su - 
# mysql
mysql> exit
```


# Command line arguments
`[]` is used for optional argument
```shell-session
$ ls [filename]
```

`...` is used for repeated arguments
```shell-session
$ ls [filename,...]
```

`<>` is used for a mandatory argument
```shell-session
$ cp <source> <destination>
```


# Program Options
Single dash `-` is followed by multiple single character options. If multiple characters are specified, each character is a specific option

The following is equivalent
```shell-session
$ ls -ltr
$ ls -l -t -r
```

A single dash `-`, when used in the place of a filename, is a synonym for `stdin`. Here we concatenate **bar** and **foo** from `stdin`
```shell-session
$ echo foo | cat bar -
```

Double dash `--` is followed by a single, multi character option
```shell-session
$ ls --all
```

A single double dash `--` without trailing characters shows the end of options. It's useful if an argument starts with a double dash
```shell-session
$ ls -al --annoying-filename
ls: unrecognized option '--annoying-filename'
Try 'ls --help' for more information.
$ ls -al -- --annoying-filename
-rw-r--r--  1 tatref tatref     0 Dec 20 22:33 --annoying-filename
```


# Sources & References
1. [POSIX - Shell Command Language](https://pubs.opengroup.org/onlinepubs/9699919799/xrat/V4_xcu_chap02.html)
1. [POSIX - Utility Conventions](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html)
1. [docopt](http://docopt.org/)
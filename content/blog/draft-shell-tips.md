+++
title = "Shell Tips"
slug = "shell-tips"
#description = "a short description"

date = 2099-01-01
#updated = 2099-01-01

draft = true

[taxonomies]
tags = ["shell"]
+++


# Introduction
The shell can be a very powerful tool, but if used like Notepad, it will just be painful and error prone

Here I present the tips I use everyday to be productive in my work


# List and Finding Files
List files by **date** with `ls -lt` (usually you will also want `-r` have most recent files at the bottom), and combine with `*`
```shell-session
$ ls -ltr messages*
-rw-r----- 1 root adm  18175 Nov 22 00:00 messages.4.gz
-rw-r----- 1 root adm 128651 Nov 29 00:17 messages.3.gz
-rw-r----- 1 root adm  93959 Dec  6 21:07 messages.2.gz
-rw-r----- 1 root adm 547103 Dec 13 17:26 messages.1
-rw-r----- 1 root adm 579873 Dec 19 21:20 messages
```

List files by **size** with `ls -lS`
```shell-session
$ ls -lS messages*
-rw-r----- 1 root adm 579873 Dec 19 21:20 messages
-rw-r----- 1 root adm 547103 Dec 13 17:26 messages.1
-rw-r----- 1 root adm 128651 Nov 29 00:17 messages.3.gz
-rw-r----- 1 root adm  93959 Dec  6 21:07 messages.2.gz
-rw-r----- 1 root adm  18175 Nov 22 00:00 messages.4.gz
```

If you're not sure what you're looking for, use `find` to search recent (`mtime -1` modify date < 24h) files (`-type f`) whose name matches some pattern (`-name "app*.log"`) with a specific size (`-size +2M`)
```shell-session
$ find /var/log -type f -mtime -1 -size +2M
```


# Repeat Last Argument
Sometime you just want to do multiple operations on the same file
```shell-session
$ vi script.sh
$ chmod +x script.sh
$ chown root: script.sh
```

The shell expansion `$!` will expand to the last argument of the last command

So you can do
```shell-session
$ vi script.sh
$ chmod +x !$
$ chown root: !$
```


# Backup or Move a File
Scared to make a mistake? Easy, just make a copy beforehand

First tip: don't name files `.back` or `.old`, always append at least the current date
```shell-session
$ cp app.conf app.conf.2020-12-19
```

Using shell expansion is much quicker
```shell-session
$ cp app.conf{,.2020-12-19}
```

Here is how it works: each part in the braces will be repeated and joined to the first argument, you can use `echo` to check
```shell-session
$ echo test{1,2,3}
test1 test2 test3
```


# Delete Files or Directories
To prevent mistakes, I tried to avoid `rm -rf` whenever possible

`rmdir` will delete an **empty** directory

It can also delete all empty parents directories up to `/` with `-p`


# Reading logs: `vi`, `tail`, `view`, `less`...?
1. Don't use `vi` or `vim` to read logs. It's safer to use `view` because it disables editing
1. `tail -f <some log>` will show use the last appended lines in the file, however, you can navigate in the file
1. `less` can combine the best of `tail -f` and `view`. Open a file with `less <log>`. You can search with `/` like in `vim` or `view`. Press `F` (SHIFT-F) to start tailing the file. Press CTRL-C to stop tailing


# Repeating a Command
If you want to check if a file gets modified, it's tempting to run `ls` multiple times
```shell-session
$ ls -l file
-rw-r--r-- 1 tatref tatref 0 Dec 19 23:07 file
$ ls -l file
-rw-r--r-- 1 tatref tatref 0 Dec 19 23:07 file
$ ls -l file
-rw-r--r-- 1 tatref tatref 0 Dec 19 23:07 file
```

Or you can just use `watch` to run the command each 2s
```shell-session
$ watch ls -l file
```

Useful flags include
* `-d`: highlight differences
* `-e`: freeze on error
* `-n <interval>`


# Sources & References
1. [TLDP - Bash Guide for "Beginners"](https://tldp.org/LDP/Bash-Beginners-Guide/html/index.html)
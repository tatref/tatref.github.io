+++
title = "Reliable Bash Scripts"
slug = "reliable-bash-scripts"
#description = "a short description"

date = 2099-01-01
#updated = 2099-01-01

draft = true

[taxonomies]
tags = ["linux", "script"]
#authors = ["JB", "Alfred", "Nobel"]
+++

# What Can Go Wrong
$ cat app.config
logpathh = /var/log/app

rm -rf $logpath

# Fail Early
set -u

set -e

# Sources & References
1. How to search stuff - [http://google.com](http://google.com)
1. Search more - [source2](http://google.com)
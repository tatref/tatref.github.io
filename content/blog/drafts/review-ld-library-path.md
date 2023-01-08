+++
title = "Shared Libraries on Linux"
slug = "2021-shared-libraries-linux"
#description = "a short description"

date = 2099-01-01
#updated = 2099-01-01

draft = true

[taxonomies]
tags = ["linux"]
+++


# Why Shared Libraries?
Here are some advantages of using shared libraries:

* Shared libraries (`.dll` on Windows, `.so` on Linux) allow code reuse between executables.
* This reduces disk usage on disk, because each executable is smaller, but also memory usage, because the library can be loaded only once by the OS, and shared between processes.
* Also, fixing bugs or security issues require updating a single package, instead of updating (or having to recompile) every dependant program.
* Last, shared libraries can be switched at run-time without recompiling the whole program, making testing easier.

However, shared libraries can prevent some optimizations opportunities from the compiler.

Shared libraries can be listed with `ldd`
```shell-session
$ ldd /usr/bin/ls
	linux-vdso.so.1 (0x00007ffedbeb9000)
	libselinux.so.1 => /lib64/libselinux.so.1 (0x00007f9104b24000)
	libcap.so.2 => /lib64/libcap.so.2 (0x00007f910491e000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f910455b000)
	libpcre2-8.so.0 => /lib64/libpcre2-8.so.0 (0x00007f91042d7000)
	libdl.so.2 => /lib64/libdl.so.2 (0x00007f91040d3000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f9104f71000)
	libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f9103eb3000)
```


# Dynamic Linker
The dynamic linker is responsible for creating the process memory image at run-time.

It's jobs are:
* copying the executable to the process image in memory
* locating and adding the required shared libraries
* transferring the execution to the program


The dynamic linker of an executable is located in the `INTERP` program header, and can be inspected with `readlelf`. On Linux, it is `/lib64/ld-linux-x86-64.so.2`
```shell-session
$ readelf -l --wide /usr/bin/ls

Elf file type is DYN (Shared object file)
Entry point 0x5e30
There are 11 program headers, starting at offset 64

Program Headers:
  Type           Offset   VirtAddr           PhysAddr           FileSiz  MemSiz   Flg Align
  PHDR           0x000040 0x0000000000000040 0x0000000000000040 0x000268 0x000268 R   0x8
  INTERP         0x0002a8 0x00000000000002a8 0x00000000000002a8 0x00001c 0x00001c R   0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  [...]
```

# Shared Libraries Locations
On Linux, the default directories containing shared libraries are `/lib`, `/usr/lib` and theirs 64 bits counter parts.

Searching theses directories on each program execution would be slow, so a cache is created at boot time with `ldconfig`. The cache is built under `/etc/ld.so.cache`, and uses the default directories, plus any additional directory defined in `/etc/ld.so.conf` or `/etc/ld.so.conf.d/`



# `LD_LIBRARY_PATH` & `LD_PRELOAD`
Sometimes it is not wise to update the `ldconfig` configuration, for example if you install a program for a single user, or if you want to test multiple versions of a library.

 The `LD_LIBRARY_PATH` and `LD_PRELOAD` environment variables can change the behavior of the dynamic linker for a single session.
 * `LD_LIBRARY_PATH`: list of comma-separated directories
 * `LD_PRELOAD`: list of comma-separated `.so` files

Here I test the loading of different versions of Oracle's Instant Client (.zip versions)

11.1 is not a supported version for this particular executable, all the other versions load correctly.

```shell-session
$ ll /u01/instant_clients/
total 8
drwxr-xr-x 2 root root  198 Apr 27 17:57 instantclient_11_1
drwxr-xr-x 2 root root  233 Apr 27 17:34 instantclient_11_2
drwxr-xr-x 2 root root 4096 Apr 27 17:34 instantclient_12_1
drwxr-xr-x 2 root root  321 Apr 27 17:34 instantclient_12_2
drwxr-xr-x 3 root root 4096 Apr 27 17:34 instantclient_18_5


$ export ORACLE_HOME=/home/oracle/LINUX.X64_193000_db_home
$ ORACLE_SID=orcl

$ for lib in /u01/instant_clients/*
> do
>   LD_LIBRARY_PATH=$lib ./target/release/orcl
> done
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: DpiError(DbError { code: 0, offset: 1269856800, message: "DPI-1050: Oracle Client library is at version 11.1 but version 11.2 or higher is needed", fn_name: "dpiContext_createWithParams", action: "check Oracle Client version" })', src/main.rs:103:44
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
Version: 11.2.0.4.0
Version: 12.1.0.2.0
Version: 12.2.0.1.0
Version: 18.5.0.0.0
```


# Debugging
The `LD_DEBUG` environment variable can be used to display dynamic linker details
```shell-session
$ LD_DEBUG=libs ls
     10442:	find library=libselinux.so.1 [0]; searching
     10442:	 search cache=/etc/ld.so.cache
     10442:	  trying file=/lib64/libselinux.so.1
     10442:	
     10442:	find library=libcap.so.2 [0]; searching
     10442:	 search cache=/etc/ld.so.cache
     10442:	  trying file=/lib64/libcap.so.2
```





# Sources & References
1. Various man pages for: `ldconfig`, `LD_LIBRARY_PATH`...
1. [Wikipedia - Dynamic Linker](https://en.wikipedia.org/wiki/Dynamic_linker)
1. [linuxfoundation.org - ELF Specification](https://refspecs.linuxfoundation.org/elf/elf.pdf)
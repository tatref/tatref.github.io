+++
title = "Python Bytecode"
slug = "2099-python-bytecode"
#description = "a short description"

date = 2099-01-01
#updated = 2099-01-01

draft = true

[taxonomies]
tags = ["test"]
+++


# Intro
## Intro 2
blabla blah

blah blah

## Un autre chapitre

Python, or more precisely, the CPython implementation, doesn't execute directly the source code.
Instead, it parses the source, and generates bytecode.
Then, the Python virtual machine executes the instructions contained in the bytecode.

The `dis` module can be used to inspect the bytecode.

It's also possible to alter the bytecode from Python itself.

# Demo
In this example, I'm using Python 3.7.3 from Debian 10. There's a few differences between Python 2 and 3, but the principles remain the same.

We define a function `f` that takes 1 argument `a`, and returns `a + 1`

```shell-session
(venv) tatref@debian:~$ python
Python 3.7.3 (default, Jul 25 2020, 13:03:44) 
[GCC 8.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> def f(a):
...     return a + 1
... 
>>> f(1)
2
>>> f(10)
11
```

We can use the `dis` module to get information on the function
```shell-session
>>> import dis
>>> dis.show_code(f)
Name:              f
Filename:          <stdin>
Argument count:    1
Kw-only arguments: 0
Number of locals:  1
Stack size:        2
Flags:             OPTIMIZED, NEWLOCALS, NOFREE
Constants:
   0: None
   1: 1
Variable names:
   0: a
```

We can also decompile the function to reveal the bytecode instructions
```shell-session
>>> dis.dis(f)
  2           0 LOAD_FAST                0 (a)
              2 LOAD_CONST               1 (1)
              4 BINARY_ADD
              6 RETURN_VALUE
```

Here is a description of the columns
1. Line number to which the instruction(s) belong to. Here, all instructions are on line 2.
1. Offset in bytes, each instruction takes up 2 bytes
1. Name of the instruction
1. Argument of the instruction, if any. 
1. Interpretation of the argument, this depend on the instruction

Each instruction is described in the [dis module docs](https://docs.python.org/3/library/dis.html#python-bytecode-instructions)


# Diving into the bytecode
Here is what the bytecode actually looks like in binary
```shell-session
>>> f.__code__.co_code.hex()
'7c00640117005300'
>>> len(f.__code__.co_code)
8
```

Instructions in binary format are called opcodes.
We already know that each instruction is 2 bytes long, but the actual opcode is only 1 byte long, the second byte is the argument.
Here is how to split the opcode and the argument

```shell-session
>>> code = f.__code__.co_code
>>> for i in range(0, int(len(code) / 2)):
...     opcode = hex(code[i*2])
...     arg    = hex(code[i*2+1])
...     print(opcode, arg)
... 
0x7c 0x0
0x64 0x1
0x17 0x0
0x53 0x0
```

It's possible to use `dis.opname` to go from opcode to instruction, and find back the instructions as with `dis.dis`.
```shell-session
>>> dis.opname[0x7c]
'LOAD_FAST'
>>> for i in range(0, int(len(code) / 2)):
...     instruction = dis.opname[code[i*2]]
...     arg         = hex(code[i*2+1])
...     print(instruction, arg)
... 
LOAD_FAST 0x0
LOAD_CONST 0x1
BINARY_ADD 0x0
RETURN_VALUE 0x0
```

Notice how the last 2 instructions have argument `0x0`?
This is just a placeholder, because these instructions don't have an argument.
`dis.HAVE_ARGUMENT` is used to check if the argument is relevant or not. In Python 2, this was a bit different: the length of instructions was variable, 

```shell-session
>>> 0x7c >= dis.HAVE_ARGUMENT
True
>>> 0x53 >= dis.HAVE_ARGUMENT
False
```


# Modifying the bytecode
```py3
code = f.__code__

new_code = types.CodeType(
        code.co_argcount,
        code.co_kwonlyargcount,
        code.co_nlocals,
        code.co_stacksize,
        code.co_flags,
        b'\x71\x03\x09\x71|\x00d\x01\x17\x00S\x00', #code.co_code, # b'|\x00d\x01\x17\x00S\x00'
        code.co_consts,
        code.co_names,
        code.co_varnames,
        code.co_filename,
        code.co_name,
        code.co_firstlineno,
        code.co_lnotab,
        code.co_freevars,
        code.co_cellvars
)       
f.__code__ = new_code
dis.dis(f)
f(1)
```

# TODO
https://github.com/tatref/misc/blob/master/python/bytecode_experiments/bytecode.py

https://pypi.org/project/trepan3k/


# Sources & References
1. https://docs.python.org/3/library/dis.html
1. https://docs.python.org/3/library/types.html


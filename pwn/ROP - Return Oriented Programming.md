#pwn 

# ¿Qué es ROP?

En la mayoría de sistemas modernos no hay necesidad de ejecutar instrucciones almacenadas en el stack, normalmente, tienen activado el [[NX bit]]. Por lo que surgió una técnica que consiste en utilizar las instrucciones ó *gadgets* del propio binario.

De esta forma, sobrescribimos  la dirección de retorno por una del stack, en la que mediante un [[Buffer overflow]], hayamos almacenado la de alguno de estos gadgets, seguida de los argumentos que más nos convengan. Cuando dicho gadget termine de ejecutarse, podemos poner la dirección de otro gadget (y encadenar tantos como sean necesarios).

Concepto relacionado e interesante ([Weird Machines](https://en.wikipedia.org/wiki/Weird_machine)).

# Técnicas

## Guardar recursos en el stack

En ocasiones necesitaremos almacenar información en el stack, como las cadenas *`/flag`* ó *`/bin/cat`*. Para ello, si conocemos su dirección en el stack, podemos incluirlas, pasando dicha dirección como parámetro, e incluso utilizando gadgets para saltarlas (como en este caso el 3 ó el 5). 

![[string_stack_ROP.png]]

Un ejemplo bueno es [[write4]].

## Janitorial gadgets (gadgets conserje)

Como hemos visto antes, a veces necesitamos gadgets para saltar datos almacenados en el stack, o realizar ciertas correcciones. Por ejemplo:

```asm
pop r12; pop rsi; ret
add rsp, 0x40; ret
```
> Estos gadgets permiten saltarnos datos en el stack, almacenándolos en registros que no estamos utilizando, o sumando al *rsp*.

## Guardar datos en registros

Con los mismos gadgets que *popean* datos en registros podemos guardar información en dichos registros:

```asm
pop rax; ret
```

Un ejemplo bueno es [[write4]].
## Guardar direcciones en registros

Los gadgets con **`lea`** que hagan lo que queremos son raros, y no suelen estar cerca de un *`ret`*. Algunas alternativas son:

```asm
push rsp; pop rax; ret    # (Equivale a mov rax, rsp; guarda la direción del stack en rax)
```

```asm
add rax, rsp; ret    # (No es perfecto, pero hace lo mismo)
```

```asm
xchg rax, rsp; ret    # (Intercambia rax y rsp, PELIGROSO)
```

Una vez tenemos la dirección del stack en un registro, podemos usarla para dinámicamente computar otras direcciones, de forma que podemos saltarnos protecciones de tipo [[ASLR]] (aunque no [[Canary|canaries]]).

Un ejemplo bueno es [[write4]].

## Pivotar el stack

En ocasiones el stack puede ser muy limitado, por lo que podemos llevárnoslo a donde queramos:

```asm
xchg rax, rsp; ret
pop rsp; ...; ret
```

Un ejemplo es [[Sick ROP HTB]].

## Transferencia de datos

Un [[Crafting shellcodes|shellcode]] necesita mover datos, puedes hacerlo con *ROPChains*:

```asm
byte [rcx], al; pop rbp; ret
```
> Aunque de esta forma se necesita un gadget que asigne *rcx*, lo cual es muy raro

## Syscalls

Es poco frecuente encontrar *syscalls*, normalmente llamarás a funciones del sistema presentes en la [[PLT]].

Un ejemplo es [[split]].

## Encontrar ROP gadgets

Hay muchas herramientas para este propósito: [[ropper]], [[rp++]], [[ROPGadget]]...

# Limitaciones

## Control limitado

En ocasiones el [[Buffer overflow]] es demasiado pequeño, obligándonos a utilizar un menor número de gadgets. Además no podemos incluir *NULLBYTES*.

### Magic gadget

Esta limitación en ocasiones podemos superarla con lo que se conoce como *magic gadget*, es un gadget que nos permite hacer una *syscall*.

## [[ASLR]]

Hay formas de bypasear esta protección, como una sobreescritura parcial de la dirección de retorno.

## [[Canary|Canaries]]

No hay una forma exacta de saltar esta protección.
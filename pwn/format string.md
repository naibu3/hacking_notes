#pwn #vuln 

# ¿Qué es?

Es una ataque que se aprovecha de la función *printf* de C. Si se permite al usuario controlar el input:

```c
printf(input);
```

Puede introducirse de manera maliciosa modificadores propios de dicha función, como *`%x`* que dejen al descubierto información del stack o incluso permitir escritura de datos.

# Explotación

## Ver registros del stack

## Escribir datos en el stack

Con el modificador *`%n`* podemos escribir el número de caracteres impresos como un entero a una dirección pasada como argumento. Con *$* podemos especificar desde dónde queremos leer esta dirección.

Por ejemplo, con la siguiente línea escribiremos *`<val>`* espacios a partir de la posición *`<pos>`*, de forma que dicho numero se guardará en la dirección *`<dir>`*:

```asm
%<dir>%<val>$n
```
> Ten en cuenta que se guardará `<val>` más la propia longitud de `<dir>`.

Con `hn` podemos escribir de dos en dos bytes (frente a los 4 de `n`).
#pwn #vuln 

# ¿Qué es?

Es una ataque que se aprovecha de la función *printf* de C. Si se permite al usuario controlar el input:

```c
printf(input);
```

Puede introducirse de manera maliciosa modificadores propios de dicha función, como *`%x`* que dejen al descubierto información del stack o incluso permitir escritura de datos.

Un dato curioso es que *printf* es ***Turing complete*** ([enlace de interés](https://github.com/HexHive/printbf)).

# Explotación

## Ver registros del stack

| Modificador | Significado                                                   |
| ----------- | ------------------------------------------------------------- |
| **`%c`**    | Lee un carácter del stack                                     |
| **`%d,%i`** | Lee un entero (4 bytes)                                       |
| **`%x`**    | Lee un entero (4 bytes) interpretándolo en hexadecimal        |
| **`%s`**    | Accede a un puntero y lo muestra hasta llegar a un *nullbyte* |

Modificadores de tamaño:

| Modificador | Significado |
| ----------- | ----------- |
| **`%x`**    | 4 bytes     |
| **`%hx`**   | 2 bytes     |
| **`%hhx`**  | 1 byte      |
| **`%lx`**   | 8 bytes     |

También es posible especificar la posición a tomar de la siguiente manera: **`%7$x`** (en este caso tomará el séptimo valor del stack)

## Escribir datos en el stack

Con el modificador *`%n`* podemos escribir el número de caracteres impresos hasta el momento, como un entero a una dirección pasada como argumento. 

```asm
%<dir>%<val>
```

Con *$* podemos especificar desde dónde queremos leer esta dirección.

Por ejemplo, con la siguiente línea escribiremos *`<val>`* espacios a partir de la posición *`<pos>`*, de forma que dicho numero se guardará en la dirección *`<dir>`*:

```asm
%<dir>%<val>$n
```
> Ten en cuenta que se guardará `<val>` más la propia longitud de `<dir>`.

Los modificadores de tamaño también se pueden utilizar:

| Modificador | Significado |
| ----------- | ----------- |
| **`%n`**    | 4 bytes     |
| **`%hn`**   | 2 bytes     |
| **`%hhn`**  | 1 byte      |
| **`%ln`**   | 8 bytes     |

### Tamaño de relleno dinámico

Sería algo así: **`%*10$c%11$n`**

Con el *asterisco* **`*`**, especificamos un tamaño dinámico de relleno, de forma que:

1. Toma el décimo parámetro (`%10`)
2. Lo utiliza como tamaño de relleno, en este caso de un sólo carácter (`$c`)
3. Imprime esa cantidad de carácteres
4. Almacena el dato en la posición de memoria indicada por la undécima posición de memoria

Osea, somos capaces de copiar el valor de una posición del stack en la dirección almacenada en otra posición 
# ¿Qué es?

Es la vulnerabilidad que se produce cuando se desbordan datos en zonas de memoria reservadas mediante funciones como `malloc`, `calloc` ó `realloc`.

## El Heap

Es un espacio destinado al almacenamiento de datos que al contrario que el stack, crece hacia posiciones altas:

![[estructura_heap.png]]

En este caso, el heap no contiene ninguna dirección de retorno, sino que al desbordar sus datos trataremos de apuntar a parámetros de las cabeceras de los *trozos* ó *chunks* que lo componen.
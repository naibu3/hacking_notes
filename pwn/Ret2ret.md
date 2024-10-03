# ¿Qué es?

Es una técnica que se basa en la forma en la que se maneja el [[Frame pointer]] al retornar de una función. Al volver, se guarda el valor de *ebp* en *esp*, de forma que si mediante un [[Buffer overflow]], somos capaces de controlar ebp, podemos sobrescribir *esp*. Y cómo *eip* es contiguo, podemos también sobrescribirlo.

El problema surge cuando existen validaciones en los datos. ó mecanismos que nos impiden saltar directamente a direcciones de funciones interesantes. Por ello, en estos casos, podemos saltar a la dirección de una instrucción `ret`, y añadir en el stack la dirección que sí nos interesa. Saltando así a dicha instrucción *ret* y evitando la posible seguridad.

Esta es la base sobre la que se construyen los ataques de tipo [[ROP - Return Oriented Programming|ROP]].
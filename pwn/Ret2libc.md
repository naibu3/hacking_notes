# ¿Qué es?

Es un ataque que consiste en hacer que el flujo de ejecución del programa salte a una función de la librería estándar de C. De esta forma, podremos llamar a funciones tan peligrosas como *system*.

De esta forma, podremos saltar protecciones como el [[NX bit]].
# Protecciones

A la hora de realizar este ataque, la protección que nos jugará más en contra es el [[ASLR]], randomizando las direcciones de las funciones de *libc*, obligándonos a complicar la explotación.

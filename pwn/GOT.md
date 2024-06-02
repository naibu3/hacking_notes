#pwn 

# ¿Qué es?

La ***Global Offset Table*** es una sección de un binario que almacena las funciones que son *dynamically linked*. Para ahorrar memoria, los programas no incluyen todas las funciones que utilizan de forma que estas funciones (por ejemplo las de *libc*), se "enlazan" al programa.

A menos que un programa sea marcado como *full [[RELRO]]*, la resolución de dirección a función en librerías dinámicas se hace de forma *lazy*. Las librerías se cargan en memoria al inicio, pero no se enlazan las funciones hasta que no se llaman por primera vez. Para conservar dichas direcciones ya utilizadas se utiliza la **GOT**.
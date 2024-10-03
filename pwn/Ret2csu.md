#pwn 

# ¿Qué es?

Es una técnica capaz de bypasear la protección de [[ASLR]], haciendo uso de los gadgets presentes en las funciones que los compiladores enlazan dinámicamente en los programas. Concretamente, hace uso de la función *`__libc_csu_init()`*.

Esta técnica sólo existe para *x86_64*.

Para más detalle lee este [paper](https://i.blackhat.com/briefings/asia/2018/asia-18-Marco-return-to-csu-a-new-method-to-bypass-the-64-bit-Linux-ASLR-wp.pdf).
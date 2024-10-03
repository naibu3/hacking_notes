#tool #python 

# ¿Qué es?

Es una librería de [[python]] que incluye una serie de herramientas y funciones orientadas a la ciberseguridad.

# Scripting

Podemos generar plantillas para nuestros scripts con:

```bash
pwn template --host <host> --port <port> <binary>
```

## Attach gdb

```
gdb.attach(<proc_name>, '''
...
continue
''')
```

# Herramientas


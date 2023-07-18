Es una herramienta en [[python]] que nos permite hacer [[port forwarding]].

# Descarga

Puedes descargar el script del siguiente [repo](https://github.com/jpillora/chisel).

# Uso

En nuestra máquina:
```bash
./chisel server -p 8000 --reverse
```

En el servidor remoto:
```bash
./chisel client 10.10.14.54:8000 R:3000:127.0.0.1:3000 R:8001:127.0.0.1:8001 &
```
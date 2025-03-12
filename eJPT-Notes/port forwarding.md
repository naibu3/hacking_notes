Mediante port forwarding, podemos hacer que un servicio que solo es accesible a través de una máquina sea accesible desde el exterior. Haciendo que puertos de nuestra máquina de atacante sean los de la máquina víctima.

# Nmap

Una vez comprometida la máquina podemos utilizarla para pivotar a una red interna.

```bash
run autoroute -s 10.0.22.69/20
```
```bash
cat /etc/proxychains4.conf # To see the port
```
```bash
background
use auxiliary/server/socks_proxy
show options
```
```
set SRVPORT 9050
set VERSION 4a 
exploit
jobs
```

# Chisel

Mediante la herramienta [[chisel]] podemos hacer port forwarding sin necesidad de privilegios.

# Ssh

Mediante [[22-ssh]] podemos hacer port forwarding:

```bash
ssh -L <puerto-local-escucha>:<host-remoto>:<puerto-remoto> <servidor-ssh>
```
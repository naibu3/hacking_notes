Mediante port forwarding, podemos hacer que un servicio que solo es accesible a través de una máquina sea accesible desde el exterior. Haciendo que puertos de nuestra máquina de atacante sean los de la máquina víctima.

# Chisel

Mediante la herramienta [[chisel]] podemos hacer port forwarding sin necesidad de privilegios.

# Ssh

Mediante [[22-ssh]] podemos hacer port forwarding:

```bash
ssh -L <puerto-local-escucha>:<host-remoto>:<puerto-remoto> <servidor-ssh>
```
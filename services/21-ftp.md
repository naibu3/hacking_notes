FTP es un protocolo ampliamente utilizado para la **transferencia de archivos** en redes. La enumeración del servicio FTP implica recopilar información relevante, como la versión del servidor FTP, la configuración de permisos de archivos, los usuarios y las contraseñas (mediante ataques de fuerza bruta o guessing), entre otros.

Para conectarse, simplemente:

```bash
ftp <ip>
```

# POC

### No existe el usuario anónimo

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
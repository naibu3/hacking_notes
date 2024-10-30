SSH (Secure Shell) es un protocolo de administración remota que permite a los usuarios **controlar** y **modificar** sus servidores remotos a través de Internet mediante un mecanismo de **autenticación seguro**. Como una alternativa más segura al protocolo **Telnet**, que transmite información sin cifrar, SSH utiliza **técnicas criptográficas** para garantizar que todas las comunicaciones hacia y desde el servidor remoto estén cifradas.

SSH proporciona un mecanismo para autenticar un usuario remoto, transferir entradas desde el cliente al host y retransmitir la salida de vuelta al cliente. Esto es especialmente útil para administrar sistemas remotos de manera segura y eficiente, sin tener que estar físicamente presentes en el sitio.

[OpenBSD SSH](https://www.openssh.com/) (`OpenSSH`) es un fork de código abierto de SSH.

Actualmente hay dos versiones de SSH: `SSH-1` y `SSH-2`. La segunda es la versión más reciente, mejor en términos de encriptación, velocidad, estabilidad y seguridad.

# Configuración

```bash
cat /etc/ssh/sshd_config  | grep -v "#" | sed -r '/^\s*$/d'
```

## Configuraciones peligrosas

| **Setting**                  | **Description**                             |
| ---------------------------- | ------------------------------------------- |
| `PasswordAuthentication yes` | Allows password-based authentication.       |
| `PermitEmptyPasswords yes`   | Allows the use of empty passwords.          |
| `PermitRootLogin yes`        | Allows to log in as the root user.          |
| `Protocol 1`                 | Uses an outdated version of encryption.     |
| `X11Forwarding yes`          | Allows X11 forwarding for GUI applications. |
| `AllowTcpForwarding yes`     | Allows forwarding of TCP ports.             |
| `PermitTunnel`               | Allows tunneling.                           |
| `DebianBanner yes`           | Displays a specific banner when logging in. |
# Auditando el servicio

## SSH-Audit

[ssh-audit](https://github.com/jtesta/ssh-audit):

```bash
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py 10.129.14.132

```

## Cambiar el método de autenticación

Podemos utilizar `PreferredAuthentications`:

```bash
ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password
```

## Versiones

Dependiendo de la versión, podríamos probar ambas versiones del protocolo SSH-1/SSH-2.

Si la versión es menor a la 7.7, podríamos tener una via potencial de comprobar usuarios válidos:

```bash
searchsploit ssh user enumeration
```

## Conexión

```bash
ssh user@ip -p <port>
```

Podemos también conectarnos proporcionando la clave *rsa* del usuario en cuestión:

```bash
ssh -i <archivo_clave> user@ip -p <port>
```

```
ssh -o PreferredAuthentications=password user@host
```

# POC

Utilizaremos el siguiente contenedor:

```bash
docker run -d \
  --name=openssh-server \
  --hostname=openssh-server `#optional` \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e PASSWORD_ACCESS=true \
  -e USER_PASSWORD=louise \
  -e USER_NAME=s4vitar `#optional` \
  -p 2222:2222 \
  -v /path/to/appdata/config:/config \
  --restart unless-stopped \
  lscr.io/linuxserver/openssh-server:latest
```

Podemos tratar de conectarnos:

```bash
ssh s4vitar@127.0.0.1 -p 2222
```

Para tratar de *bruteforcear* la contraseña:

```bash
hydra -l s4vitar -P rockyou.txt ssh://127.0.0.1 -s 2222 -t 15
```
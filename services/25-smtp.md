---
title:
  "{ title }": 
service: TCP
---
# ¿Qué es?

`Simple Mail Transfer Protocol` (`SMTP`) es un servicio de transferencia de e-mails. Normalmente se combina con IMAP ó POP3 qué permiten buscar emails y enviarlos.

Por defecto corre sobre el puerto *25*, aunque versiones más recientes pueden operar además por otros puertos como el *587*. Éstos puertos secundarios se utilizan al principio de la comunicación para el login, siendo el resto de la comunicación en texto plano. Para evitar acceso no autorizado a la información que se envía, se puede utilizar encriptación *SSL/TLS*, pasando la comunicación a otro puerto como el *465*.

Una función esencial de un servidor SMTP es prevenir el spam mediante mecanismos de autenticación que sólo permitan el envío a usuarios autenticados. Para ello, los servidores modernos permiten la extensión ESMTP con SMTP-Auth.

Después de enviar el email, el cliente ó `Mail User Agent` (`MUA`), lo convierte en un *header* y un *body* y lo manda al servidor. El servidor dispone de un `Mail Transfer Agent` (`MTA`) que checkea el email en cuanto a su tamaño y si es spam. Previamente puede pasar por un `Mail Submission Agent` (`MSA`) o *Relay*. Finalmente en el destino, el `Mail delivery agent` (`MDA`) vuelve a ensamblar el email.

| Client (`MUA`) | `➞` | Submission Agent (`MSA`) | `➞` | Open Relay (`MTA`) | `➞` | Mail Delivery Agent (`MDA`) | `➞` | Mailbox (`POP3`/`IMAP`) |
| -------------- | --- | ------------------------ | --- | ------------------ | --- | --------------------------- | --- | ----------------------- |

Sin embargo, SMTP tiene dos grandes desventajas:

1. No devuelve una confirmación de entrega útil. Aunque si que existe una confirmación, no existe una estándar, por lo que suele ser un mensaje en inglés.
2. Cuando la conexión se establece los usuarios aún no están autenticados, lo que ha sido ampliamente atacado. Hoy día existen medidas como [Sender Policy Framework](https://dmarcian.com/what-is-spf/) (`SPF`).

Para mitigar estas problemáticas se creó `Extended SMTP` (`ESMTP`), qué es la versión más utilizada a día de hoy.

# Configuración

## Configuración por defecto

```bash
cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"
```
```shell-session
smtpd_banner = ESMTP Server 
biff = no
append_dot_mydomain = no
readme_directory = no
compatibility_level = 2
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
myhostname = mail1.inlanefreight.htb
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
smtp_generic_maps = hash:/etc/postfix/generic
mydestination = $myhostname, localhost 
masquerade_domains = $myhostname
mynetworks = 127.0.0.0/8 10.129.0.0/16
mailbox_size_limit = 0
recipient_delimiter = +
smtp_bind_address = 0.0.0.0
inet_protocols = ipv4
smtpd_helo_restrictions = reject_invalid_hostname
home_mailbox = /home/postfix
```

El envío y la comunicación se hace a través de comando especiales:

|**Command**|**Description**|
|---|---|
|`AUTH PLAIN`|AUTH is a service extension used to authenticate the client.|
|`HELO`|The client logs in with its computer name and thus starts the session.|
|`MAIL FROM`|The client names the email sender.|
|`RCPT TO`|The client names the email recipient.|
|`DATA`|The client initiates the transmission of the email.|
|`RSET`|The client aborts the initiated transmission but keeps the connection between client and server.|
|`VRFY`|The client checks if a mailbox is available for message transfer.|
|`EXPN`|The client also checks if a mailbox is available for messaging with this command.|
|`NOOP`|The client requests a response from the server to prevent disconnection due to time-out.|
|`QUIT`|The client terminates the session.|

## Configuraciones peligrosas

Antes hemos mencionado el uso de *Relay* servers, en este caso, si encontramos una configuración tal que:

```shell-session
mynetworks = 0.0.0.0/0
```

El servidor no sólo aceptará emails del relay, sino de cualquier otro origen, permitiendo a un atacante falsificar correos o *spoofear* los correos.

# Auditando el servicio

## Conexión

Para conectarnos y realizar la comunicación, podemos utilizar [[telnet]] y utilizamos `HELO` ó `EHLO` para iniciar la comunicación:

```bash
telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server 


HELO mail1.inlanefreight.htb

250 mail1.inlanefreight.htb


EHLO mail1

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
```

### Enumeración de usuarios

```shell-session
VRFY root

252 2.0.0 root


VRFY cry0l1t3

252 2.0.0 cry0l1t3


VRFY testuser

252 2.0.0 testuser
```

Aunque no siempre será fiable.

### Proxy

En caso de estar utilizándose un proxy podemos especificarlo con `CONNECT 10.129.14.128:25 HTTP/1.0`.ç

### Enviar un email

```bash
MAIL FROM: <cry0l1t3@inlanefreight.htb>

250 2.1.0 Ok


RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure

250 2.1.5 Ok


DATA

354 End data with <CR><LF>.<CR><LF>

From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200
Hey man, I am trying to access our XY-DB but the creds don't work. 
Did you make any changes there?
.

250 2.0.0 Ok: queued as 6E1CF1681AB


QUIT
```

En las cabeceras de un email viaja mucha información que podría ser de utilidad para un atacante. La especificación de la estructura de un email se recoge en [RFC5322](https://datatracker.ietf.org/doc/html/rfc5322).

## Nmap

[[nmap]] lanza por defecto scripts de enumeración para SMTP. Sin embargo, podemos lanzar el script de NSE [smtp-open-relay](https://nmap.org/nsedoc/scripts/smtp-open-relay.html) para ver si el servidor tiene esta vuln.

## smtp-user-enum

[[smtp-user-enum]] es una herramienta que como su nombre indica, permite buscar usuarios válidos en SMTP.

## [[metasploit]]

```bash
scanner/smtp/smtp_enum
```


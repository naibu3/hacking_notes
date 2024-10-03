---
service: TCP
---
# ¿Qué es?

`Internet Message Access Protocol` (`IMAP`) permite acceder a los emails de un servidor de email, permitiendo su edición en el propio servidor y una vista de subcarpetas. 

Por otro lado, `Post Office Protocol` (`POP3`), sólo permite listar correos, devolverlos ó eliminarlos.

Dependiendo de si se usa encriptación TLS/SSL la comunicación de IMAP puede ser por el puerto `143` ó `993`, y para POP3 por el `110` ó `995`.

# Configuración

## IMAP Commands

| **Command**                     | **Description**                                                                                               |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `1 LOGIN username password`     | User's login.                                                                                                 |
| `1 LIST "" *`                   | Lists all directories.                                                                                        |
| `1 CREATE "INBOX"`              | Creates a mailbox with a specified name.                                                                      |
| `1 DELETE "INBOX"`              | Deletes a mailbox.                                                                                            |
| `1 RENAME "ToRead" "Important"` | Renames a mailbox.                                                                                            |
| `1 LSUB "" *`                   | Returns a subset of names from the set of names that the User has declared as being `active` or `subscribed`. |
| `1 SELECT INBOX`                | Selects a mailbox so that messages in the mailbox can be accessed.                                            |
| `1 UNSELECT INBOX`              | Exits the selected mailbox.                                                                                   |
| `1 FETCH <ID> all`              | Retrieves data associated with a message in the mailbox.                                                      |
| `1 CLOSE`                       | Removes all messages with the `Deleted` flag set.                                                             |
| `1 LOGOUT`                      | Closes the connection with the IMAP server.                                                                   |

## POP3 Commands

|**Command**|**Description**|
|---|---|
|`USER username`|Identifies the user.|
|`PASS password`|Authentication of the user using its password.|
|`STAT`|Requests the number of saved emails from the server.|
|`LIST`|Requests from the server the number and size of all emails.|
|`RETR id`|Requests the server to deliver the requested email by ID.|
|`DELE id`|Requests the server to delete the requested email by ID.|
|`CAPA`|Requests the server to display the server capabilities.|
|`RSET`|Requests the server to reset the transmitted information.|
|`QUIT`|Closes the connection with the POP3 server.|

## Configuraciones peligrosas

| **Setting**               | **Description**                                                                           |
| ------------------------- | ----------------------------------------------------------------------------------------- |
| `auth_debug`              | Enables all authentication debug logging.                                                 |
| `auth_debug_passwords`    | This setting adjusts log verbosity, the submitted passwords, and the scheme gets logged.  |
| `auth_verbose`            | Logs unsuccessful authentication attempts and their reasons.                              |
| `auth_verbose_passwords`  | Passwords used for authentication are logged and can also be truncated.                   |
| `auth_anonymous_username` | This specifies the username to be used when logging in with the ANONYMOUS SASL mechanism. |

# Auditando el servicio

## Nmap

Como siempre, [[nmap]] dispone de una serie de scripts básicos de enumeración para este servicio.

## cURL

[[curl]] puede ser de utilidad para interactuar con el servicio:

```bash
curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd
```

## OpenSSL / TLS 

En caso de utilizar encriptación SSL/TLS podemos operar de la siguiente forma:

```bash
openssl s_client -connect 10.129.14.128:pop3s
```
> Para POP3

```bash
openssl s_client -connect 10.129.14.128:imaps
```
> Para IMAP

Una vez conectados a IMAPS podemos buscar información:

```
1 LOGIN robin robin
1 lIST “” *
1 SELECT DEV.DEPARTMENT.INT
1 fetch 1 all
```

Puedes ver todos los comandos de IMAPS [aquí](https://www.atmail.com/blog/imap-commands/?source=post_page-----5e5c99547f8a--------------------------------).
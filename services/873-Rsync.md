# ¿Qué es?

[Rsync](https://linux.die.net/man/1/rsync) es una herramienta eficiente para copiar archivos localmente o de forma remota, conocida por su algoritmo de transferencia delta, que solo envía las diferencias entre archivos. Se usa para respaldos y espejeo, y por defecto opera en el puerto *873*, pudiendo configurarse con SSH para mayor seguridad.

# Auditando el servicio

## [[nmap]]

```bash
sudo nmap -sV -p 873 127.0.0.1
```

## Buscar recursos accesibles

```bash
nc -nv 127.0.0.1 873
```

___
# R-Services

Los R-Services son un conjunto de servicios que permiten acceso remoto o ejecución de comandos entre hosts Unix a través de TCP/IP. Fueron desarrollados por el CSRG de la Universidad de California, Berkeley, y fueron el estándar para acceso remoto en sistemas Unix hasta que fueron reemplazados por SSH debido a sus vulnerabilidades de seguridad. Al igual que telnet, los R-Services transmiten información sin cifrar, lo que los hace susceptibles a ataques de intermediario (MITM).

Operan en los puertos *512*, *513* y *514*, y se acceden mediante r-commands. Aunque son menos comunes hoy en día, todavía aparecen en pruebas de penetración interna.

Los [R-commands](https://en.wikipedia.org/wiki/Berkeley_r-commands) son:

- rcp (`remote copy`)
- rexec (`remote execution`)
- rlogin (`remote login`)
- rsh (`remote shell`)
- rstat
- ruptime
- rwho (`remote who`)

|**Command**|**Service Daemon**|**Port**|**Transport Protocol**|**Description**|
|---|---|---|---|---|
|`rcp`|`rshd`|514|TCP|Copy a file or directory bidirectionally from the local system to the remote system (or vice versa) or from one remote system to another. It works like the `cp` command on Linux but provides `no warning to the user for overwriting existing files on a system`.|
|`rsh`|`rshd`|514|TCP|Opens a shell on a remote machine without a login procedure. Relies upon the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files for validation.|
|`rexec`|`rexecd`|512|TCP|Enables a user to run shell commands on a remote machine. Requires authentication through the use of a `username` and `password` through an unencrypted network socket. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files.|
|`rlogin`|`rlogind`|513|TCP|Enables a user to log in to a remote host over the network. It works similarly to `telnet` but can only connect to Unix-like hosts. Authentication is overridden by the trusted entries in the `/etc/hosts.equiv` and `.rhosts` files.|
## /etc/hosts.equiv

El archivo **/etc/hosts.equiv** contiene una lista de hosts confiables y se utiliza para otorgar acceso a otros sistemas en la red. Los usuarios de estos hosts pueden acceder al sistema automáticamente sin necesidad de autenticación adicional.

# Auditando R-Services

## [[nmap]]

```bash
sudo nmap -sV -p 512,513,514 10.0.17.2
```

## Login con rlogin

El principal problema de los r-services, y una de las razones por las que [[22-ssh|SSH]] fue introducido para reemplazarlos, son las debilidades en el control de acceso. R-services confían en la información enviada por el cliente remoto para autenticarlo en el host. Aunque utilizan módulos PAM para autenticar usuarios en el sistema remoto, este proceso puede ser omitido mediante los archivos **/etc/hosts.equiv** y **.rhosts**, que contienen listas de hosts y usuarios confiables. Estos archivos permiten que se establezcan conexiones sin autenticación adicional mediante r-commands. Las entradas en estos archivos pueden incluir direcciones IP o nombres de host.

```bash
cat .rhosts

htb-student     10.0.17.5
+               10.0.17.10
+               +
```

```bash
rlogin 10.0.17.2 -l htb-student
```

## Listar usuarios

```bash
rwho
```

```bash
rusers -al 10.0.17.5
```
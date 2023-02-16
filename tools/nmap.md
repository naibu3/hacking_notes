Nmap es una potente herramienta para el escaneo de redes y hosts, y recopilación de información.


## Escaneo de puertos

Mediante un escaneo de puertos podemos determinar que puertos TCP y UDP están abiertos y qué servicios y versiones corren, con tal de encontrar vulns y fallas de seguridad.
Para ello utilizamos el parámetro `-p`:

```bash
nmap -p- <ip>
nmap -p1,23,445,... <ip>
nmap -p100-2000 <ip>
nmap -p- <ip>
nmap -p- <ip>
```

## Modos de escaneo

- #### Ping sweeping

Con `-sn` podemos lanzar n,ap para que escanee un rango de direcciones en busca de hosts activos:

```bash
nmap -sn 200.200.0.0/16
nmap -sn 200.200.123.1-23
nmap -sn 192.123.34.*
nmap -sn 200.200.1-12.*
```

Ó tomando los rangos de un archivo con `-iL` (*input list*):

```bash
nmap -sn -iL archivo.txt
```

Cabe destacar que en ocasiones puede pasar que un firewall bloquee las peticiones haciendo parecer que un host no se encuentre activo, para estas ocasiones es conveniente buscar hosts mediante escaneo de puertos, lo que se detalla en [[Host discovery]].

- #### OS finguerprinting
 
Para que el escaneo trate de reportarnos el SO de la máquina objetivo mediante [[OS finguerprinting]], utilizaremos el parámetro `-O`:

```bash
nmap -O <ip>
```

Con `--osscan-limit` podemos poner un limite al escaneo en caso de que haya que escanear una gran cantidad de hosts. Ó `--osscan-guess` para un escaneo más agresivo.

- #### TCP connect scan

Se basa en el funcionamoiento del [[TCP three way handshake]], ya que trata de establecer una conexión, si el servidor la acepta (*SYN+ACK*), el puerto está abierto, por lo que una vez conectado manda un paquete (*SYN+RST*); en caso de que la rechace (*RST+ACK*), el puerto está cerrado.

El problema de este modo es que todos los paquetes quedan registrados en los logs de la máquina objetivo, de forma que es muy fácil de detectar.

Se especifica con `-sT`:

```bash
nmap -sT <ip>
```

- #### TCP SYN scan

Este modo es más sigiloso ya que en lugar de completar la conexión, tan sólo envía un primer paquete *SYN*, y analiza la respuesta sin completar la conexión.

Se especifica con `-sS`:

```bash
nmap -sS <ip>
```

- #### Detección de versiones

Con el parámetro `-sV`, nmap tratará de detectar lasversiones de los servicios que corren en cada puerto.

```bash
nmap -sV <ip>
```

En caso de que el host bloquee el reconocimiento de versiones, podemos ver la razón con `--reason`.

- #### Firewalls

Para esos casos en los que un firewall bloquee los pings, haciendo parecer que el host no esté activo podemos utilizar el parámetro `-Pn`, para que no se aplique *host discovery*, es decir para que trate el host como si estuviera activo:

```bash
nmap -Pn <ip>
```

Como hemos explicado antes, el servidor puede bloquear la búsqueda de versiones, por lo que podemos ver qué lo ha bloqueado con `--reason`.

## Scripts de nmap

- #### Scripts de búsqueda generales

A la hora de buscar vulns podemos lanzar scripts básicos de enumeración, con el parámetro `-sC`, ó utilizar el script `vuln`, que buscará todo tipo de vulns.
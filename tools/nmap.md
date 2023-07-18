---
title: nmap
author: naibu3
---

#reconocimiento 

# nmap

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

Se basa en el funcionamoiento del [[TCP three way handshake]], ya que trata de establecer una conexión, si el servidor la acepta (*SYN+ACK*), el puerto está abierto, por lo que una vez conectado manda un paquete (*SYN+RST*); en caso de que la rechace (*RST+ACK*), el puerto está cerrado. Es el escaneo por defecto en *nmap*.

El problema de este modo es que todos los paquetes quedan registrados en los logs de la máquina objetivo, de forma que es muy fácil de detectar.

Se especifica con `-sT`:

```bash
nmap -sT <ip>
```

- #### TCP SYN scan

Este modo es más sigiloso ya que en lugar de completar la conexión, tan sólo envía un primer paquete *SYN*, y analiza la respuesta sin completar la conexión. Esto en ocasiones puede ser más sigiloso a nivel de *logs*, además de más rápido.

Se especifica con `-sS`:

```bash
nmap -sS <ip>
```

- #### UDP scan

Este modo permite hacer un escaneo de los puertos que hacen uso del protocolo *UDP*.

Se especifica con `-sU`:

```shell
nmap -sU <ip>
```

- #### Mejorar velocidad del escaneo

En general es recomendable utilizar el parámetro `-n` ya que en caso de aplicarse resolución DNS los escaneos se demoran más tiempo. Igualmente, también es recomendable el uso de `-Pn` para que del mismo modo no se aplique descubrimiento de host.

Para mejorar la velocidad en los escaneos *TCP connect* se suelen utilizar las plantillas de temporizado (parámetro desde `-T0` a `-T5`), siendo el 0 el modo más lento y 5 el más rápido. 

Por otro lado, en los *TCP SYN*, se suele utilizar el `--min-rate` con un valor de 5000, para que no se envíen paquetes más lentos que 5000 por segundo.

- #### Detección de versiones

Con el parámetro `-sV`, nmap tratará de detectar lasversiones de los servicios que corren en cada puerto.

```bash
nmap -sV <ip>
```

En caso de que el host bloquee el reconocimiento de versiones, podemos ver la razón con `--reason`.

- #### Firewalls

Para esos casos en los que un *firewall* bloquee los *pings*, haciendo parecer que el host no esté activo podemos utilizar el parámetro `-Pn`, para que no se aplique *host discovery*, es decir para que trate el host como si estuviera activo:

```bash
nmap -Pn <ip>
```

Como hemos explicado antes, el servidor puede bloquear la búsqueda de versiones, por lo que podemos ver qué lo ha bloqueado con `--reason`.

La primera forma de evadir un *firewall* es mediante la **fragmentación de los paquetes**. Muchos firewalls esperan un tipo de paquete concreto, al fragmentarlos rompes ese esquema, evadiendo la seguridad. Esto puede realizarse con el parámetro `-f`.

Otra forma es mediante el *mtu*, especificando el parámetro `--mtu <valor>`, donde dicho valor debe ser múltiplo de 8.

En los casos en los que ciertas *IPs* de origen estén bloqueadas ó haya un máximo de peticiones por *IP*, podríamos utilizar el *decoy*, y con el parámetro `-D <decoy>`, especificar las distintas *IP* bajo las que se enviarán los paquetes.

De igual forma, en el caso de los puertos de origen, con el parámetro `--source-port`, podemos especificar un puerto a través del cuál realizaremos el escaneo.

Por otro lado, con `--data-length`, podemos añadir un extra de longitud a los paquetes.

Además podemos falsificar la dirección MAC de origen con el parámetro `--spoof-mac <OUI>`
permitiendo especificar una OUI.

Finalmente, el uso de `-sS` (tcp SYN scan), en determinadas ocasiones puede ser mejor ya que no deja logs.

## Scripts de nmap

- #### Scripts de búsqueda generales

A la hora de buscar vulns podemos lanzar scripts básicos de enumeración, con el parámetro `-sC`, ó utilizar el la opción `--script` con la categoría y el script especifíco que queremos lanzar.

Por ejemplo:

```shell
nmap <ip> --script="vuln"
nmap <ip> --script="vuln or/and safe"
nmap <ip> --script <nombre del script>
```

- #### Categorias

Los scripts están organizados en las siguientes categorías:

-   `auth`: scripts relacionados con la autenticación de usuarios
-   `broadcast`: en esta categoría se engloban scripts que utilizan peticiones de transmisión para recopilar información de los host que no se listan por broadcasting
-   `brute`: esta categoría es para scripts que utilizan el sistema de fuerza bruta para averiguar las credenciales de usuario en un determinado servicio
-   `default`: estos son los scripts que se ejecutan cuando se ejecuta la opción -sC
-   `discovery`: scripts relacionados con el descubrimiento de servicios y hosts
-   `dos`: esta categoría es para scripts relacionados con ataques de denegación de servicio
-   `exploit`: esta categoría es para scripts que explotan vulnerabilidades de seguridad
-   `external`: esta categoría es para scripts que utilizan datos o servicios de terceros
-   `fuzzer`: esta categoría es para scripts NSE que se centran en fuzzing (envían campos inesperados o aleatorios en cada paquete)
-   `intrusive`: esta categoría es para scripts que pueden bloquear algo o generar mucho ruido en la red
-   `malware`: scripts relacionados con la detección de malware
-   `safe`: esta categoría es para scripts que se consideran seguros en todas las situaciones
-   `version`: esta categoría contiene scripts que extienden la funcionalidad de la detección de versiones
-   `vuln`: esta categoría es para scripts relacionados con vulnerabilidades de seguridad

Para ver las categorías:

```shell
locate .nse | xargs grep "categories" | grep -oP ".*?" | sort -u
```

Podemos buscarlos con el comando *locate*:

```shell
locate .nse
```


- ##### Fuzzear subdominios

```nmap
nmap --script dns-brute -n <ip>
```

- ##### Enumerar http

```nmap
nmap --script http-enum -n <ip>
```

- ##### Estructura de un script

```lua
-- HEAD --
description=[[
Regla de ejemplo que enumera y reportalos puertos abiertos por TCP
]]

-- RULE --
portrule = function(host, port)
	return port.protocol == "tcp"
		and port.state == "open"
end

-- ACTION --
action = function(host, port)
	return "This port is open!"
end
```
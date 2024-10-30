---
title: nmap
author: naibu3
---
#reconocimiento 
# ¿Qué es?

Network Mapper (`Nmap`) es una potente herramienta *open-source*, escrita en C, C++, Python y Lua para el escaneo de redes y hosts, y recopilación de información.

Entre sus usos está:

- Host discovery
- Port scanning
- Enumeración y detección de servicios
- Detección de SO
- Interacción con el objetivo mediante scripts (Nmap Scripting Engine)

# Host discovery

Con `-sn` podemos lanzar nmap para que escanee un rango de direcciones en busca de hosts activos:

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

Otra opción es utilizar `ICMP Echo requests` en lugar de los `ARP ping`, para ello podemos lanzar la herramienta con el parámetro `-PE`, ó desactivando directamente ARP con `--disable-arp-ping`:

```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace
```
> `--packet-trace` nos permite ver todos los paquetes que se envían y reciben.

Otro parámetro interesante es `--reason` que nos permite ver el porqué nmap ha marcado un host como "*alive*"
# Escaneo de puertos

Mediante un escaneo de puertos podemos determinar que puertos TCP y UDP están abiertos y qué servicios y versiones corren, con tal de encontrar vulnerabilidades y fallas de seguridad.

Para especificar el rango de puertos utilizamos el parámetro `-p`:

```bash
nmap -p- <ip>
nmap -p1,23,445,... <ip>
nmap -p100-2000 <ip>
nmap -p- <ip>
nmap -p- <ip>
nmap -F #Top 100
nmap --top-ports=10
```

Los puertos escaneados nos reportarán un estado:

| **State**          | **Description**                                                                                                                                 |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `open`             | Nos indica que la conexión se ha establecido, por tanto está abierto (puede ser por **conexión TCP**, **datagramas UDP** ó **asociación SCTP**) |
| `closed`           | Cuando el estado es cerrado, significa que la respuesta contiene la flag `RST`. Por tanto el puerto está cerrado.                               |
| `filtered`         | Como no se recibe respuesta ó es un error, se clasifica con este estado.                                                                        |
| `unfiltered`       | Ocurre en un escaneo **TCP-ACK** y significa que el puerto es accessible, pero no se sabe si está abierto o cerrado.                            |
| `open\|filtered`   | Si no obtenemos respuesta, se nos da este estado, lo que puede revelar la existencia de un firewall o filtro de paquetes.                       |
| `closed\|filtered` | Solo ocurre en escaneos de tipo **IP ID idle** y nos indica que fue imposible determinar si el puerto está cerrado o protegido por un firewall. |

## Modos de escaneo
 
- ### TCP connect scan

Es el modo de escaneo de Nmap para usuarios no privilegiados,

Es el modo más certero para comprobar si un puerto está abierto. Funciona completando el *Three-way handshake* y marca un puerto abierto si recibe el paquete `SYN-ACK` y como cerrado si recibe `RST`. Además, es también el más sigiloso ya que no deja conexiones a medias.

Puede incluso bypasear ciertos firewalls que solo bloquean paquetes entrantes, aunque es uno de los modos más lentos, ya que tiene que esperar los paquetes de confirmación.

Se especifica con `-sT`:

```bash
nmap -sT <ip>
```

- ### TCP SYN scan

Es el modo por defecto en caso de lanzar Nmap con privilegios.

Este modo es más sigiloso ya que en lugar de completar la conexión, tan sólo envía un primer paquete *SYN*, y analiza la respuesta sin completar la conexión. Esto en ocasiones puede ser más sigiloso a nivel de *logs*, además de más rápido.

Se especifica con `-sS`:

```bash
sudo nmap -sS <ip>
```

Para ver más claramente el escaneo y mejorar la velocidad, podemos desactivar los *ICMP echo requests* (`-Pn`), la *resolución DNS* (`-n`), y el *ARP ping scan* (`--disable-arp-ping`).

- ### UDP scan

Este modo permite hacer un escaneo de los puertos que hacen uso del protocolo *UDP*.

Se especifica con `-sU`:

```shell
nmap -sU <ip>
```

## Detección de versiones

Con el parámetro `-sV`, nmap tratará de detectar las versiones de los servicios que corren en cada puerto.

```bash
nmap -sV <ip>
```

En caso de que el host bloquee el reconocimiento de versiones, podemos ver la razón con `--reason`.

Con `-sC` podemos mandar también que se ejecuten los scripts básicos de reconocimiento, pudiendo compactar ambos parámetros en `-sCV`.

### Banner grabbing

La forma en la que Nmap recoge la información es mediante la lectura de los banners que envían los servicios. En ocasiones, Nmap no sabe cómo procesar dicha información, y por tanto, no nos la representa. Para ello podemos utilizar otras herramientas como [[tcpdump]]:

```bash
sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28
```

También [[netcat]]:

```bash
nc -nv 10.129.2.28 25
```

## Performance

En general es recomendable utilizar el parámetro `-n` ya que en caso de aplicarse *resolución DNS* los escaneos se demoran más tiempo. Igualmente, también es recomendable el uso de `-Pn` para que del mismo modo no se aplique *descubrimiento de host*. Otro parámetro interesante es el que desactiva el *ARP ping scan* (`--disable-arp-ping`).

Para mejorar la velocidad en los escaneos *TCP connect* se suelen utilizar las plantillas de temporizado (parámetro desde `-T0` a `-T5`), siendo el 0 el modo más lento y 5 el más rápido. 

Por otro lado, en los *TCP SYN*, se suele utilizar el `--min-rate` con un valor de 5000, para que no se envíen paquetes más lentos que 5000 por segundo.

Otros parámetros que pueden mejorar la performance son:

- `--min-parallelism <number>`
- `--max-rtt-timeout <time>`, cuando nmap envía un paquete se le asigna un *Round-Trip-Time* ó *RTT* que suele estar asignado a 100ms.
- `--max-retries <number>`

### Ver información

Para listar más información durante y después del escaneo, podemos utilizar desde `-v` a `-vvv`, ó `--stats-every=5s` para que nos reporte información cada cierto intervalo.

# Evasión de medidas de seguridad

## Firewalls

Para esos casos en los que un *firewall* (software que filtra las conexiones entrantes) bloquee los *pings*, haciendo parecer que el host no esté activo podemos utilizar el parámetro `-Pn`, para que no se aplique *host discovery*, es decir para que trate el host como si estuviera activo:

```bash
nmap -Pn <ip>
```

Como hemos explicado antes, el servidor puede bloquear la búsqueda de versiones, por lo que podemos ver qué lo ha bloqueado con `--reason`.

La primera forma de evadir un *firewall* es mediante la **fragmentación de los paquetes**. Muchos firewalls esperan un tipo de paquete concreto, al fragmentarlos rompes ese esquema, evadiendo la seguridad. Esto puede realizarse con el parámetro `-f`.

Otra forma es mediante el *mtu*, especificando el parámetro `--mtu <valor>`, donde dicho valor debe ser múltiplo de 8.

De igual forma, en el caso de los puertos de origen, con el parámetro `--source-port`, podemos especificar un puerto a través del cuál realizaremos el escaneo.

Por otro lado, con `--data-length`, podemos añadir un extra de longitud a los paquetes.

Además podemos falsificar la dirección MAC de origen con el parámetro `--spoof-mac <OUI>`
permitiendo especificar una OUI.

Finalmente, el uso de `-sS` (tcp SYN scan), en determinadas ocasiones puede ser mejor ya que no deja logs.

## IDS/IPS

IDS son las siglas de *Intrusion Detection System*, e IPS, *Intrusion Prevention System*. Son también piezas de software que permiten enviar una alerta en caso de que se detecte una intrusión ó intento de intrusión. Generalmente los IPS son un complemento de los IDS.

Estos sistemas son mucho más difíciles de detectar por los atacantes que los firewalls. Y normalmente, al detectar un atacante, banearán su IP impidiéndole el acceso a la red.

### Decoys

Para estos casos en los que ciertas *IPs* de origen estén bloqueadas ó haya un máximo de peticiones por *IP*, podríamos utilizar el *decoy*, y con el parámetro `-D <decoy>`, especificar las distintas *IP* bajo las que se enviarán los paquetes.

```bash
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

También podemos tratar de especificar la IP de origen con `-S <IP>`, ya que muchas máquinas en ocasiones sólo son accesibles desde determinadas subredes.

## Proxies DNS

A veces las compañías poseen servidores DNS propios con niveles de confianza mayores a la hora de acceder a hosts internos. Nmap nos da la opción de especificar servidores DNS, `--dns-server <ns>,<ns>`.

En ocasiones bastará con simplemente cambiar el puerto de origen con `--source-port`.

```bash
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

# Guardar resultados

Podemos exportar los resultados en 3 formatos diferentes:

- Normal output (`-oN`) con la extensión `.nmap`
- Grepeable output (`-oG`) con la extensión `.gnmap`
- XML output (`-oX`) con la extensión `.xml`

# Detección de SO

Para que el escaneo trate de reportarnos el SO de la máquina objetivo mediante [[Fingerprinting]], utilizaremos el parámetro `-O`:

```bash
nmap -O <ip>
```

Con `--osscan-limit` podemos poner un limite al escaneo en caso de que haya que escanear una gran cantidad de hosts. Ó `--osscan-guess` para un escaneo más agresivo.

# Scripting de nmap

## Scripts de búsqueda generales

A la hora de buscar vulns podemos lanzar scripts básicos de enumeración, con el parámetro `-sC`, ó utilizar el la opción `--script` con la categoría y el script especifíco que queremos lanzar.

Por ejemplo:

```shell
nmap <ip> --script="vuln"
nmap <ip> --script="vuln or/and safe"
nmap <ip> --script <nombre del script>
```

## Categorias

Los scripts están organizados en las siguientes categorías:

| **Category** | **Description**                                                                                                                                       |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `auth`       | scripts relacionados con la autenticación de usuarios                                                                                                 |
| `broadcast`  | en esta categoría se engloban scripts que utilizan peticiones de transmisión para recopilar información de los host que no se listan por broadcasting |
| `brute`      | esta categoría es para scripts que utilizan el sistema de fuerza bruta para averiguar las credenciales de usuario en un determinado servicio          |
| `default`    | estos son los scripts que se ejecutan cuando se ejecuta la opción `-sC`                                                                               |
| `discovery`  | scripts relacionados con el descubrimiento de servicios y hosts                                                                                       |
| `dos`        | esta categoría es para scripts relacionados con ataques de denegación de servicio                                                                     |
| `exploit`    | esta categoría es para scripts que explotan vulnerabilidades de seguridad                                                                             |
| `external`   | esta categoría es para scripts que utilizan datos o servicios de terceros                                                                             |
| `fuzzer`     | esta categoría es para scripts que se centran en fuzzing (envían campos inesperados o aleatorios en cada paquete)                                     |
| `intrusive`  | esta categoría es para scripts que pueden bloquear algo o generar mucho ruido en la red                                                               |
| `malware`    | scripts relacionados con la detección de malware                                                                                                      |
| `safe`       | esta categoría es para scripts que se consideran seguros en todas las situaciones                                                                     |
| `version`    | esta categoría contiene scripts que extienden la funcionalidad de la detección de versiones                                                           |
| `vuln`       | esta categoría es para scripts relacionados con vulnerabilidades de seguridad                                                                         |

Para ver las categorías:

```shell
locate .nse | xargs grep "categories" | grep -oP ".*?" | sort -u
```

Podemos buscarlos con el comando *locate*:

```shell
locate .nse
```

## Scripts específicos

- ### Fuzzear subdominios

```nmap
nmap --script dns-brute -n <ip>
```

- ### Enumerar http

```nmap
nmap --script http-enum -n <ip>
```

## Estructura de un script

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
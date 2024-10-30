---
title: Host discovery
---

Cuando tratamos con una red que desconocemos debemos comenzar con la fase de descubrimiento de host ó *host discovery*. En ésta fase trataremos de buscar todos aquellos host que se encuentren activos.


## Host discovery con ping

Una de los métodos más utilizados es mediante la utilidad [[ping]], que mediante herramientas como [[fping]] ó [[nmap]], lanzaremos contra cada dirección dentro del rango de direcciones de la red, llevando a cabo un barrido de direcciones (*sweeping*). También podemos tratar de montarnos nuestro propio script:

```bash
#!/bin/bash

for i in $(seq 2 254); do

	timeout 1 bash -c "ping -c 1 x.x.x.$i > /dev/null 2>&1" && echo "Host x.x.x.$i - ACTIVE" &

done; wait
```

>Con este script lanzamos un bucle que lanzará un hilo para comprobar cada host en el rango de la red.

Una vez descubiertos todos los hosts, podemos aplicar [[Fingerprinting]] para recopilar un poco más de información.


## Arp-scan

[[arp-scan]] es una buena herramienta para ver que equipos hay conectados a una red.

## Masscan

Una buena herramienta para escanear redes completas en busca de hosts es [[masscan]], que si bien es un poco más imprecisa que [[nmap]], es más potente. 


## Busqueda de hosts por escaneo de puertos

En ocasiones, un firewall puede bloquear las peticiones con ping, causando que el host parezca inactivo. En estos casos, lo más recomandable es realizar una búsqueda mediante el escaneo de los puertos más comunes (22, 445, 80, 443). 
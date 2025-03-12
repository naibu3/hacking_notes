---
title: NetBIOS
service: TCP
---
# ¿Qué es?

Es una API que permite interacción entre aplicaciones de diferentes máquinas Windows. Con este protocolo, podemos tener redes con distintos hosts, que se agrupan en diferentes *workgroups*. IBM desarrolló una API para redes de ordenadores denominada *Network Basic Input/Output System* (`NetBIOS`).

Está relacionado con 3 servicios principalmente:

- Name Service (`NetBIOS-NS:137`) - Permite a los usuarios registrar y resolver nombres de dominio.
- Datagram Service (`NetBIOS-DGM:138`) - Soporta comunicación no orientada a conexión y broadcasting.
- Session Service (`NetBIOS-SSN:139`) - Soporta comunicaciones orientadas a conexión.

En este tipo de redes, cuando un host entra a la red necesita un nombre, que se asigna mediante el procedimiento de registro de nombres. El propio host puede asignar su nombre ó puede utilizarse el [NetBIOS Name Server](https://networkencyclopedia.com/netbios-name-server-nbns/) (`NBNS`). También existe una versión mejorada, el [NetBIOS Name Server](https://networkencyclopedia.com/netbios-name-server-nbns/) (`NBNS`).

# Configuración

# Auditando el servicio

## [[nmap]]

```bash
nmap -sU -p 137 -sV -T4 --script nbstat.nse -Pn -n <ip>
```

## [[nbtscan]]

```bash
nbtscan <ip>
```

## [[nmbloockup]]

```bash
nmbloockup -A <ip>
```
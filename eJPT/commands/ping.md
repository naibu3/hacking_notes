---
title: curl
shell: bash powershell
---

Es una utilidad diseñada para comprobar si una máquina se encuentra disponible en la red. Funciona enviando una paquete ICMP especial (Type 8-***echo request***) a un host, si éste responde con otro paquete ICMP (***echo reply***), entonces el host está disponible.

Existen herramientas que automatizan el barrido del rango de direcciones para encontrar hosts, como [[fping]] ó [[nmap]].
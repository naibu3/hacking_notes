---
title: Pivoting
---
# ¿Qué es?

Es la operación de acceder a una *red interna* saltando a través de una máquina de la red.

# Metasploit

```bash
ifconfig

run autoroute -s 10.2.24.200 255.255.240.0
```

Una vez escaneemos la máquina a la que hemos pivotado, debemos añadir un [[port forwarding]] para cada puerto abierto:

```bash
sessions -i 1
portfwd add -l 1234 -p 80 -r demo2.ine.local
portfwd list
```

# Ligolo

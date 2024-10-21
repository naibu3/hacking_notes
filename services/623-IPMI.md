# ¿Qué es?

[Intelligent Platform Management Interface](https://www.thomas-krenn.com/en/wiki/IPMI_Basics) (`IPMI`) es un servicio que permite gestionar dispositivos incluso apagados ó después de un fallo. Actúa como un subsistema autónomo a parte del sistema matriz.

# Configuraciones peligrosas

## Contraseñas por defecto

| Product         | Username      | Password                                                                  |
| --------------- | ------------- | ------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                    |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
| Supermicro IPMI | ADMIN         | ADMIN                                                                     |

Si las credenciales por defecto fallan en un BMC, se puede explotar una vulnerabilidad en el protocolo RAKP de IPMI 2.0. Esto permite obtener hashes de contraseñas de cualquier cuenta válida y descifrarlas con [[hashcat]] (modo **7300**). En HP iLO, se puede usar un ataque de máscara para probar combinaciones de letras mayúsculas y números en contraseñas de ocho caracteres (`hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u`).

# Auditando el servicio

IPMI trabaja sobre el puerto `623/UDP`. Los sistemas que implementan IPMI necesitan *Baseboard Management Controllers* (BMCs) que suelen ser chips ARM con linux, aunque también existen placas de expansión PCI. Los BMCs más comunes son *HP iLO*, *Dell DRAC*, y *Supermicro IPMI*.

La mayoría de sistemas IPMI exponen una consola de configuración web, un servicio de acceso remoto por línea de comandos como [[22-ssh|ssh]] ó [[telnet]], y el puerto 623 UDP.

## Nmap

Con [[nmap]] podemos lanzar el script `ipmi-version`:

```shell-session
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

## Metasploit

Con [[metasploit]] podemos lanzar el módulo [IPMI Information Discovery (auxiliary/scanner/ipmi/ipmi_version)](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_version/):

```shell-session
use auxiliary/scanner/ipmi/ipmi_version 
```

También se puede utilizar el módulo [IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval](https://www.rapid7.com/db/modules/auxiliary/scanner/ipmi/ipmi_dumphashes/) para sacar los hashes:

```shell-session
use auxiliary/scanner/ipmi/ipmi_dumphashes 
```
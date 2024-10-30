---
title: WHOIS
---
# ¿Qué es?

**WHOIS** es un protocolo que permite consultar quién posee o administra dominios, direcciones IP y otros recursos de internet.

Un registro WHOIS típicamente incluye la siguiente información:

- **Domain Name**: El nombre del dominio (ej. example.com).
- **Registrar**: La empresa donde se registró el dominio (ej. GoDaddy, Namecheap).
- **Registrant Contact**: La persona u organización que registró el dominio.
- **Administrative Contact**: La persona encargada de gestionar el dominio.
- **Technical Contact**: La persona que maneja problemas técnicos del dominio.
- **Creation and Expiration Dates**: Fecha de registro y de expiración del dominio.
- **Name Servers**: Servidores que traducen el nombre de dominio a una dirección IP.

## Historia

La historia de WHOIS está relacionada con [Elizabeth Feinler](https://en.wikipedia.org/wiki/Elizabeth_J._Feinler), una científica informática que fue clave en el desarrollo del internet. En los años 70, su equipo en el NIC del Instituto de Investigación de Stanford creó el directorio WHOIS para gestionar el creciente número de recursos en ARPANET, almacenando información sobre usuarios, nombres de host y dominios.

# Instalación

```bash
sudo apt update
sudo apt install whois -y
```

# Uso

```bash
whois inlanefreight.com
```
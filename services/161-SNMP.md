---
title: SNMP
service: UDP
---
# ¿Qué es?

`Simple Network Management Protocol` ([SNMP](https://datatracker.ietf.org/doc/html/rfc1157)), es un protocolo creado para monitorizar dispositivos de red como switches, routers o incluso dispositivos IoT. Actualmente se encuentra en su versión `SNMPv3`.

Además de información, este protocolo puede enviar comandos, sobre el puerto `161/UDP`. SNMP también incluye `traps` sobre el puerto `162/UDP`, estos son paquetes que el servidor manda al cliente sin que se le hayan pedido explícitamente.

## MIB

Para que SNMP funcione con diferentes fabricantes y diferentes combinaciones cliente-servidor, se creó la `Management Information Base` (`MIB`). Un MIB es un fichero de objetos SNMP organizados jerárquicamente de forma que puedan buscarse. Deben contener al menos un `Object Identifier` (`OID`) y se escriben en `Abstract Syntax Notation One` (`ASN.1`). No contienen datos pero sí la estructura. 

## OID

Representan un nodo en la estructura. Tienen un identificador único.

## Versiones

### SNMPv1

Es la primera versión y ofrece las funcionalidades básicas, por lo que se sigue utilizando en gran cantidad de sistemas a día de hoy. Si embargo, *no tiene ningún mecanismo autenticación* por defecto, permitiendo a cualquiera entrar e interactuar con el servicio. Además, *no soporta encriptación*.

### SNMPv2

La versión que continúa a día de hoy es `v2c`, la `c` es de *community-based*. Sin embargo, tiene un problema al iniciar, ya que la `community string` que proporciona seguridad sólo se transmite en texto plano.

### SNMPv3

Mejora la seguridad de las versiones anteriores, aunque también aumenta la complejidad.

# Configuración

```bash
cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'
```
```shell-session
sysLocation    Sitting on the Dock of the Bay
sysContact     Me <me@example.org>
sysServices    72
master  agentx
agentaddress  127.0.0.1,[::1]
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1
rocommunity  public default -V systemonly
rocommunity6 public default -V systemonly
rouser authPrivUser authpriv -V systemonly
```

## Configuraciones peligrosas

| **Settings**                                     | **Description**                                                                       |
| ------------------------------------------------ | ------------------------------------------------------------------------------------- |
| `rwuser noauth`                                  | Provides access to the full OID tree without authentication.                          |
| `rwcommunity <community string> <IPv4 address>`  | Provides access to the full OID tree regardless of where the requests were sent from. |
| `rwcommunity6 <community string> <IPv6 address>` | Same access as with `rwcommunity` with the difference of using IPv6.                  |

# Auditando el servicio

## SNMPwalk

[[SNMPwalk]] es una herramienta que permite obtener los OIDs con su información:

```bash
snmpwalk -v2c -c public 10.129.14.128
```

## OneSixtyOne

En caso de no reconocer la *community-string*, podemos utilizar [[OneSixtyOne]] con SecLists:

```bash
onesixtyone -c /opt/useful/SecLists/Discovery/SNMP/snmp.txt 10.129.14.128
```

## Braa

Una vez conocemos la *community-string*, podemos utilizar [[Braa]] para aplicar fuerza-bruta y sacar los OIDs individuales:

```bash
braa <community string>@<IP>:.1.3.6.*
```
# ¿Qué es?

*Domain Name System* (`DNS`) es el servicio encargado de resolver nombres de dominio en IPs. Existen varios tipos de servidores DNS:

| **Server Type**                | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DNS Root Server`              | The root servers of the DNS are responsible for the top-level domains (`TLD`). As the last instance, they are only requested if the name server does not respond. Thus, a root server is a central interface between users and content on the Internet, as it links domain and IP address. The [Internet Corporation for Assigned Names and Numbers](https://www.icann.org/) (`ICANN`) coordinates the work of the root name servers. There are `13` such root servers around the globe. |
| `Authoritative Nameserver`     | Authoritative name servers hold authority for a particular zone. They only answer queries from their area of responsibility, and their information is binding. If an authoritative name server cannot answer a client's query, the root name server takes over at that point.                                                                                                                                                                                                            |
| `Non-authoritative Nameserver` | Non-authoritative name servers are not responsible for a particular DNS zone. Instead, they collect information on specific DNS zones themselves, which is done using recursive or iterative DNS querying.                                                                                                                                                                                                                                                                               |
| `Caching DNS Server`           | Caching DNS servers cache information from other name servers for a specified period. The authoritative name server determines the duration of this storage.                                                                                                                                                                                                                                                                                                                             |
| `Forwarding Server`            | Forwarding servers perform only one function: they forward DNS queries to another DNS server.                                                                                                                                                                                                                                                                                                                                                                                            |
| `Resolver`                     | Resolvers are not authoritative DNS servers but perform name resolution locally in the computer or router.                                                                                                                                                                                                                                                                                                                                                                               |

DNS per se es un servicio sin encriptación, lo que permite a un atacante interceptar los paquetes. Para ello se crearon `DNS over TLS` (`DoT`), `DNS over HTTPS` (`DoH`) ó `DNSCrypt`. Además, un servidor DNS almacena información sobre los hosts y sus servicios.

![[DNS.png]]

En las queries DNS se utilizan diferentes *DNS records*:

|**DNS Record**|**Description**|
|---|---|
|`A`|Returns an IPv4 address of the requested domain as a result.|
|`AAAA`|Returns an IPv6 address of the requested domain.|
|`MX`|Returns the responsible mail servers as a result.|
|`NS`|Returns the DNS servers (nameservers) of the domain.|
|`TXT`|This record can contain various information. The all-rounder can be used, e.g., to validate the Google Search Console or validate SSL certificates. In addition, SPF and DMARC entries are set to validate mail traffic and protect it from spam.|
|`CNAME`|This record serves as an alias for another domain name. If you want the domain www.hackthebox.eu to point to the same IP as hackthebox.eu, you would create an A record for hackthebox.eu and a CNAME record for www.hackthebox.eu.|
|`PTR`|The PTR record works the other way around (reverse lookup). It converts IP addresses into valid domain names.|
|`SOA`|Provides information about the corresponding DNS zone and email address of the administrative contact.|

# Configuración

Existen muchos tipos de configuraciones para los servidores DNS. Pero principalmente se utilizan 3 ficheros:

1. ficheros de configuración DNS locales
2. ficheros de zona
3. ficheros de resolución reversa de nombres ("*reverse name resolution files*")

El servidor DNS [Bind9](https://www.isc.org/bind/) es el más utilizado en distribuciones Linux. Sus archivos de configuración local `named.conf`, se suelen dividir en dos secciones: la primera con configuraciones generales y la segunda con entradas de zona para dominios individuales. Los archivos en cuestión suelen ser:

- `named.conf.local`
- `named.conf.options`
- `named.conf.log`

Debemos diferenciar entre opciones globales y locales (las primeras aplicadas a todas las zonas y las segundas para una única zona).

## Configuración DNS local

```bash
cat /etc/bind/named.conf.local

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
zone "domain.com" {
    type master;
    file "/etc/bind/db.domain.com";
    allow-update { key rndc-key; };
};

```

En este archivo podemos definir las diferentes zonas. Cada zona se divide en un archivo que suele contener un único dominio.

### Zone file

Son los archivos que definen una zona bajo el formato BIND. Deben tener un *SOA record* y al menos un *NS record*.

```bash
cat /etc/bind/db.domain.com

;
; BIND reverse data file for local loopback interface
;
$ORIGIN domain.com
$TTL 86400
@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

      IN     NS     ns1.domain.com.
      IN     NS     ns2.domain.com.

      IN     MX     10     mx.domain.com.
      IN     MX     20     mx2.domain.com.

             IN     A       10.129.14.5

server1      IN     A       10.129.14.5
server2      IN     A       10.129.14.7
ns1          IN     A       10.129.14.2
ns2          IN     A       10.129.14.3

ftp          IN     CNAME   server1
mx           IN     CNAME   server1
mx2          IN     CNAME   server2
www          IN     CNAME   server2
```

### Reverse Name Resolution Zone Files

Son necesarios para que un `Fully Qualified Domain Name` (`FQDN`) sea resuelto a una dirección utilizando un *PTR record*.

```bash
cat /etc/bind/db.10.129.14

;
; BIND reverse data file for local loopback interface
;
$ORIGIN 14.129.10.in-addr.arpa
$TTL 86400
@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. (
                    2001062501 ; serial
                    21600      ; refresh after 6 hours
                    3600       ; retry after 1 hour
                    604800     ; expire after 1 week
                    86400 )    ; minimum TTL of 1 day

      IN     NS     ns1.domain.com.
      IN     NS     ns2.domain.com.

5    IN     PTR    server1.domain.com.
7    IN     MX     mx.domain.com.
...SNIP...
```

## Configuraciones peligrosas

|**Option**|**Description**|
|---|---|
|`allow-query`|Defines which hosts are allowed to send requests to the DNS server.|
|`allow-recursion`|Defines which hosts are allowed to send recursive requests to the DNS server.|
|`allow-transfer`|Defines which hosts are allowed to receive zone transfers from the DNS server.|
|`zone-statistics`|Collects statistical data of zones.|

# Auditando el servicio

## Dig

Una de las herramientas que podemos utilizar es [[dig]], para mandar DNS records:

```shell-session
dig ns inlanefreight.htb @10.129.14.128
```

```bash
dig CH TXT version.bind 10.129.120.85
```

```bash
dig any inlanefreight.htb @10.129.14.128
```

# Transferencia de zona (zone transfer)

Se refiere a transferencias de zona hacia otro servidor DNS, lo que generalmente ocurre a través del puerto `53/TCP`. Éste procedimiento se abrevia como *Asynchronous Full Transfer Zone* (`AXFR`). Se realiza para asegurar que todas las zonas tienen datos idénticos, utilizando una clave `rndc-key` para asegurar la transferencia.

El servidor que contiene los datos originales de una zona se conoce como primario. A su vez, existen servidores secundarios para descargar y dar disponibilidad y seguridad al primero. Para algunos `Top-Level Domains` (`TLDs`), es obligatorio hacer accesibles los archivos de zona en al menos dos servidores para los `Second Level Domains`.

Normalmente, las modificaciones se hacen en los servidores primarios.

Un atacante puede tratar de solicitar la información del servidor mediante una transferencia de zona, dando lugar a un [[Ataques de transferencia de zona|ataque de transferencia de zona]]. Esto podemos hacerlo con [[dig]]:

```
dig axfr inlanefreight.htb @10.129.14.128
```

Mediante fuerza bruta, también podemos conseguir los registros A con los hostnames:

```bash
for sub in $(cat /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done
```

Ó podemos hacerlo con herramientas como [[DNSenum]]:

```bash
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```


---
title: Unified HTB
date: 2023-04-27
tier: starting-point
author: naibu3
---

#log4unify #linux 

# Unified HTB

Es una de las máquinas del *starting-point* de *Hack The Box*.

## Reconocimiento

Como siempre comenzamos lanzando [[nmap]], para tratar de buscar puertos abiertos:

```bash
nmap -p- --open -T5 -n 10.129.169.218 -v
```
```nmap
PORT     STATE SERVICE
22/tcp   open  ssh
6789/tcp open  ibm-db2-admin
8080/tcp open  http-proxy
8443/tcp open  https-alt
8843/tcp open  unknown
8880/tcp open  cddbp-alt
```

Buscamos ahora las versiones y servicios:

```bash
nmap -sCV -p22,6789,8080,8443,8843,8880 10.129.169.218
```
```nmap
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
6789/tcp open  ibm-db2-admin?
8080/tcp open  http-proxy
| fingerprint-strings: 
[...]
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Did not follow redirect to https://10.129.169.218:8443/manage
8443/tcp open  ssl/nagios-nsca Nagios NSCA
| http-title: UniFi Network
|_Requested resource was /manage/account/login?redirect=%2Fmanage
| ssl-cert: Subject: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US
| Subject Alternative Name: DNS:UniFi
| Not valid before: 2021-12-30T21:37:24
|_Not valid after:  2024-04-03T21:37:24
8843/tcp open  ssl/unknown
| fingerprint-strings: 
[...]
| ssl-cert: Subject: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US
| Subject Alternative Name: DNS:UniFi
| Not valid before: 2021-12-30T21:37:24
|_Not valid after:  2024-04-03T21:37:24
8880/tcp open  cddbp-alt?
| fingerprint-strings: 
[...]
3 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
[...]
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Explotación



## Escalada de privilegios


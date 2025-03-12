# Windows Access Tokens

Son un elemento clave del proceso de autenticación. Son creadas por el *Local Security Authority Subsystem Service* (**LSASS**). Es una especie de *cookie* que permite autenticar usuarios. Todos los procesos creados por un usuario con cierto access token mantendrán dicho token.

# Explotación

## Requisitos

La explotación dependerá de los privilegios de la cuenta con la que se ha ganado acceso a la máquina. Para un ataque exitoso se requiere alguno de los siguientes privilegios:

- `SeAssignPrimaryToken`
- `SeCreateToken`
- `SeImpersonatePrivilege`

## Incognito (meterpreter)

Es un módulo de [[metasploit|meterpreter]] que nos permite realizar el ataque.

```meterpreter
load incognito

list_tokens -u

impersonate_token "token"
```

## Potato Attack

Cuando no tenemos tokens disponibles, podemos utilizar el *potato attack*.
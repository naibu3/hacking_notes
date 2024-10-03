Es una herramienta que nos permite conectarnos a Samba ([[445-smb]]) desde la terminal de Linux.

# Uso

## Listar recursos

```bash
smbclient -L \\\\<ip> -N
```

```bash
smbclient \\\\10.129.42.253\\users
```

- `-N` se utiliza cuando no disponemos de credenciales.
- `-L` se utiliza para enumerar los recursos compartidos disponibles en el servidor SMB o Samba.
- `-U` se utiliza para especificar el nombre de usuario y la contraseña utilizados para la autenticación con el servidor SMB o Samba.
- `-c` se utiliza para especificar un comando que se ejecutará en el servidor SMB o Samba.

En caso de lograr acceder podemos utilizar `ls` para listar archivos y directorios.

## Descargar recursos

Para descargar un recurso en nuestra máquina, podemos utilizar `get`.
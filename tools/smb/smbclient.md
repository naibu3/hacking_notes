Es una herramienta quenos permite conectarnos a Samba ([[445-smb]]) desde la terminal de linux.

# Uso

```bash
smbclient -L <ip> -N
```

- `-N` se utiliza cuando no disponemos de credenciales.
- `-L` se utiliza para enumerar los recursos compartidos disponibles en el servidor SMB o Samba.
- `-U` se utiliza para especificar el nombre de usuario y la contraseña utilizados para la autenticación con el servidor SMB o Samba.
- `-c` se utiliza para especificar un comando que se ejecutará en el servidor SMB o Samba.
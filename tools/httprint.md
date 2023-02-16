Es una herramienta de [[Web Server finguerprinting]] ó reconocimiento de servidores web, que nos ayuda a extraer información relevante de un servidor.

## Uso

La sintaxis más utilizada sería:

```bash
httprint -P0 -h <hosts> -s <signature-file>
```

- `-P0` evita hacer ping a la máquina (muchas no responden a las peticiones *ping echo*).
- `-h` especifica el ó los hosts.
- `-s` especifica el archivo de firma.
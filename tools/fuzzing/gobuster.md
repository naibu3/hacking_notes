Es otra herramienta de fuzzing ([[Directories and files enumeration 1]]) similar a [[dirb 1]] o [[dirbuster]]. Esta escrita en *go*, por lo que trabaja muy bien con *sockets* y conexiones.

## Uso

# Para listar directorios/archivos:

```bash
gobuster dir -u <url> -w <wordlist> -t 100
```
> `-t` es para especificar el número de hilos.

En ocasiones es necesario el `--add-slash`, para que añada una `/` a las direcciones. Con `-b` podemos ocultar ciertos códigos de estado y de igual forma, con `-s` mostrarlos.

Si quisiéramos buscar archivos, con `-x` podemos *fuzzear* añadiendo una cierta extensión.

# Buscar subdominios:

```
gobuster dns --domain inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -t 200
```

```bash
gobuster vhost -u http://only4you.htb/ -w /usr/share/wordlists/dirb/common.txt -t 10 --append-domain
```
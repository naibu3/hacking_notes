---
title: Subdomain enumeration
---
Uno de los pasos más importantes durante el reconocimiento es identificar todos los sitios web que posee la organización a auditar. Muchas organizaciones tienen un dominio principal ó *top-level domain*, mediante la búsqueda de subdominios podemos identificar otros recursos del sistema.

# Identificación pasiva

Es decir, aquella que se realiza sin interactuar directamente con el objetivo, sino tan sólo mediante fuentes abiertas. Una primera forma sería utilizando [[Google Dorks]]:

```google
site: domain.com
```

Otra herramienta sería [dnsdumpster](https://dnsdumpster.com/), que utiliza datos de google, así como de otros sitios como [virustotal](https://virustotal.com).

Además hay otras herramientas para la terminal, como [[sublist3r]], (viene preinstalado con kali, pero puedes instalarlo desde su [github](https://github.com/aboul3la/Sublist3r)). Ésta toma información de DNS, pero debe utilizarse con cuidado ya que google suele bloquearla. O [[amass]].

# Identificación activa

## Certificado SSL

Lo primero que podemos hacer es ver el certificado SSL, que normalmente suele utilizarse para varios subdominios al mismo tiempo. Algunas herramientas que automatizan el proceso son [crt.sh](https://crt.sh) ó [CTFR]().

Estas herramientas utilizan el standard de *Certificate Transparency* que guarda la información en una base de datos pública para asegurar la legitimidad de los certificados. Podemos acceder a dicha información de la siguiente manera:

```bash
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .
```

Para filtrar por los propios subdominios:

```bash
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

## Fuzzing

Trata de utilizar fuerza bruta para buscar subdominios, los mejores diccionarios son los de [seclists](https://github.com/danielmiessler/SecLists). Algunas son:

- ### Gobuster

La herramienta [[gobuster]] con el modo *vhost*.

```bash
gobuster vhost -u <dominio> -w <wordlist> -t <no de hilos> --append-domain
```

```bash
gobuster vhost -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u board.htb -t 200 --append-domain
```

- ### Wfuzz

Es una herramienta específica de *fuzzing*, por lo que es más cómoda:

```bash
wfuzz -c -t <no de hilos> -w <wordilist> -H "Host: FUZZ.dominio" <url> --hc=403
```
> Con `-c` se muestra la salida con colores. Con `--hc` podemos ocultar un cierto código http ó con `--sc` mostrarlo.

- ### Sublist3r

La herramienta [[sublist3r]] es una herramienta que utiliza *OSINT*.

```bash
python3 sublist3r.py -d <dominio>
```

### Diccionarios

```bash
/usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```
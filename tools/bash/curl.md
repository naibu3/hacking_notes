---
title: curl
shell: bash
---

## Descripción

*Curl* nos permite hacer una petición a una página web. Mostrándonos la respuesta por pantalla.

## Uso

```bash
curl <url>
```


### Tramitar una petición

```bash
curl -s -X METHOD <url>
```
> `-s` no muestra el output de http.

#### Utilizando un proxy

```bash
curl <url> --proxy <proxy>
```

#### Pasar cabeceras o data

```bash
curl <url> -H <headers> -d <data>
```

### Imprimir código de respuesta de una página.

En ocasiones queremos comprobar si una página existe o no, es decir el código de respuesta, para ello, utilizaremos:

```bash
curl --write-out "%{http_code}" --output /dev/null --silent --insecure <url>
```

Con `--write-out "%{http_code}"` haremos que imprima el código de respuesta, después con `--output` redirigiremos la salida html al */dev/null*, con `--silent` evitaremos que se imprima info. de contexto del propio comando, y , finalmente con `--insecure`, evitaremos que se imprima la advertencia en caso de estar utilizando http y no http**s**.
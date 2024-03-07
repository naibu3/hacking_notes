---
title: wfuzz
author: naibu3
---

# wfuzz

## Ocultar resultados

En ocasiones, las peticiones fallidas responderán con códigos de respuesta que harán que se reporten, podemos ocultarlos con la opción `--hw <value>`

## Subdomain fuzzing

```bash

```

## Directories/files fuzzing

```bash
wfuzz -c --hc=404,403 -t 200 -w <wordlist> http://dominio/FUZZ/
```
> `-c` es para que se muestre la salida con colores.
> `-t` es para especificar el número de hilos.
> `--hc` es para ocultar cierto código de estado.
> `-L` hace que siga las redirecciones (aunque es mejor añadir una barra directamente).
> `--sl` permite mostrar respuestas con un número específico de líneas.

Podemos fuzzear también base a un rango:

```bash
wfuzz -c --hc=404,403 -t 200 -z range,1-20000 http://dominio/?id=FUZZ
```

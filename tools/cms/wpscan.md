wpscan es una herramienta que nos permite analizar un [[Wordpress]].

# Uso

```bash
wpscan --url <direccion>
```

Si deseas enumerar usuarios en WordPress, puedes añadir los siguientes parámetros a la línea de comandos:

```bash
wpscan --url https://example.com --enumerate u
```

En caso de querer enumerar plugins existentes los cuales sean vulnerables, puedes añadir el siguiente parámetro a la línea de comandos:

```bash
wpscan --url https://example.com --enumerate vp
```

Scan installed plugins

wpscan --url http(s)://your-domain.com --enumerate p

Scan vulnerable plugins

wpscan --url http(s)://your-domain.com --enumerate vp

Scan installed themes

wpscan --url http(s)://your-domain.com --enumerate t

Scan vulnerable themes

wpscan --url http(s)://your-domain.com --enumerate vt

Scan user accounts:

wpscan --url http(s)://your-domain.com --enumerate u

Scan vulnerable timthumb files:

wpscan --url http(s)://your-domain.com --enumerate tt
# Api token

Puedes conseguirlo [aquí](https://wpscan.com/api). No permitirá obtener más información del escaneo:

```bash
wpscan --url https://example.com --api-token <token>
```
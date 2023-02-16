*Hypertext Transfer Protocol* (HTTP) ó Protocolo de Transferencia de Hipertexto es un protocolo de la **capa de aplicación** para la transmisión de documentos hipermedia, como HTML. Fue diseñado para la comunicación entre los navegadores y servidores web, aunque se puede utilizar para otros propósitos también.

## HTTP Verbs

- ### GET

Se utiliza para hacer una petición de un recurso.

> Ejemplo:
```http
GET /resource HTTP/1.1
Host: www.host.site
```
> Ejemplo con un argumento:
```http
GET /resource?argument=value HTTP/1.1
Host: www.host.site
```

- ### HEAD

Es muy similar a GET, pero solo pide las cabeceras de la respuesta, sin incluir el cuerpo.

> Ejemplo:
```http
HEAD /resource HTTP/1.1
Host: www.host.site
```

- ### POST

Se utiliza para enviar datos de un formulario HTML. Los parámetros deben ir en el cuerpo del mensaje:

> Ejemplo:
```http
POST /resource.php HTTP/1.1
Host: www.host.site

param=hola&param2=adios
```

- ### PUT

Se utiliza para subir un archivo, lo que puede ser muy peligroso si está mal configurado.

> Ejemplo:
```http
PUT /path/to/resource HTTP/1.1
Host: www.host.site

<put data>
```

- ### DELETE

Se utiliza para borrar un archivo, lo que también puede ser muy peligroso si está mal configurado.

> Ejemplo:
```http
DELETE /path/to/resource HTTP/1.1
Host: www.host.site
```

- ### OPTIONS

Se utiliza para ver qué *HTTP Verbs* están habilitados.

> Ejemplo:
```http
OPTIONS / HTTP/1.1
Host: www.host.site
```


## REST APIs

Las *Representational State Transfer APIs* son un tipo específico de aplicaciones que se basa fuertemente en casi todos los HTTP Verbs. Se suelen llamar *web services* o simplemente *APIs*. Debido a que se apoyan en los *HTTP Verbs* no sería de extrañar que suelan contener funcionalidades potencialmente vulnerables.

Es muy común en estas aplicaciones que se utilice PUT para guardar datos y no archivos. Por tanto, antes de reportar un método PUT ó DELETE debemos identificar si realmente sube un archivo ó simplemente crea contenido nuevo. Lo más fiable, una vez subido el supuesto archivo, sería tratar de buscarlo.
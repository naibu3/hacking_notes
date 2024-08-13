---
author: naibu3
---
# Content discovery

Cuando auditamos una página, debemos buscar contenido, que si bien es accesible, no debería serlo. Por ejemplo, imágenes/videos, páginas de staff, versiones antiguas, archivos de configuración...

Hay varios métodos para realizarlo: manualmente, de forma automática o mediante [[OSINT]].

## Manualmente

### Robots.txt

Es un archivo que dice a las arañas de los buscadores qué páginas no indexar, por lo que suele contener rutas interesantes.

### Favicon

Es el pequeño icono que aparece en el navegador. En ocasiones tenemos acceso al favicon del [[CMS]] sobre el que corre la aplicación, para ello podemos buscarlo en páginas como esta de [OWASP](https://wiki.owasp.org/index.php/OWASP_favicon_database).

Si lo descargamos y obtenemos el *hash md5* (`md5sum`), podemos saber qué [[CMS]] se está utilizando.

### Sitemap.xml

Al contrario que el *robots.txt*, este archivo indica las rutas que deben indexarse.

### HTTP Headers

En las *cabeceras http* normalmente podemos encontrar información acerca de las tecnologías que se utilizan.

```bash
curl http://10.10.59.41 -v
```

## OSINT

Podemos aplicar [[OSINT]] para encontrar contenidos.

Una técnica común cuando trabajamos con **código abierto**, es buscar en los propios repositorios de [[GitHub]] del proyecto *Changelogs* que nos den pistas sobre posibles versiones de la aplicación.
## Automatizada

Podemos automatizar esta búsqueda con herramientas de **[[fuzzing]]** como:

- [[ffuf]]
- [[wfuzz]]
- [[dirb]]
- [[dirbuster]]
- [[gobuster]]
- ...
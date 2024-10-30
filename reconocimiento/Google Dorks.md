Una buena forma de enumerar recursos en un servidor es utilizando sentencias de búsqueda avanzada en google, ó *Google Dorks*. Estas sentencias son una combinación especial de comandos de google que permiten encontrar recursos especiales en webs.

```python
inurl:admin intitle:login
```
> Ejemplo para encontrar un panel de login.

## Comandos interesantes


| Operador        | Descripción del Operador                                                | Ejemplo                                 | Descripción del Ejemplo                                                                                 |
|-----------------|-------------------------------------------------------------------------|-----------------------------------------|----------------------------------------------------------------------------------------------------------|
| `site:`         | Limita los resultados a un sitio web o dominio específico.              | `site:example.com`                      | Encuentra todas las páginas públicamente accesibles en example.com.                                      |
| `inurl:`        | Encuentra páginas que contengan un término específico en la URL.        | `inurl:login`                           | Busca páginas de inicio de sesión en cualquier sitio web.                                                |
| `filetype:`     | Busca archivos de un tipo particular.                                   | `filetype:pdf`                          | Encuentra documentos PDF descargables.                                                                   |
| `intitle:`      | Encuentra páginas que contengan un término específico en el título.     | `intitle:"confidential report"`         | Busca documentos titulados "confidential report" o variaciones similares.                                |
| `intext:` o `inbody:` | Busca un término dentro del texto del cuerpo de las páginas.     | `intext:"password reset"`               | Identifica páginas web que contengan el término "password reset".                                        |
| `cache:`        | Muestra la versión en caché de una página web (si está disponible).     | `cache:example.com`                     | Visualiza la versión en caché de example.com para ver su contenido anterior.                             |
| `link:`         | Encuentra páginas que enlazan a una página específica.                  | `link:example.com`                      | Identifica sitios web que enlazan a example.com.                                                         |
| `related:`      | Encuentra sitios web relacionados con una página específica.            | `related:example.com`                   | Descubre sitios web similares a example.com.                                                             |
| `info:`         | Proporciona un resumen de información sobre una página web.             | `info:example.com`                      | Obtén detalles básicos sobre example.com, como su título y descripción.                                  |
| `define:`       | Proporciona definiciones de una palabra o frase.                        | `define:phishing`                       | Obtén una definición de "phishing" de varias fuentes.                                                    |
| `numrange:`     | Busca números dentro de un rango específico.                            | `site:example.com numrange:1000-2000`   | Encuentra páginas en example.com que contengan números entre 1000 y 2000.                                |
| `allintext:`    | Encuentra páginas que contengan todas las palabras especificadas en el cuerpo del texto. | `allintext:admin password reset` | Busca páginas que contengan tanto "admin" como "password reset" en el cuerpo del texto.                 |
| `allinurl:`     | Encuentra páginas que contengan todas las palabras especificadas en la URL. | `allinurl:admin panel`            | Busca páginas con "admin" y "panel" en la URL.                                                          |
| `allintitle:`   | Encuentra páginas que contengan todas las palabras especificadas en el título. | `allintitle:confidential report 2023` | Busca páginas con "confidential," "report," y "2023" en el título.                                       |
| `AND`           | Restringe los resultados exigiendo que estén presentes todos los términos. | `site:example.com AND (inurl:admin OR inurl:login)` | Busca páginas de administración o inicio de sesión específicamente en example.com.  |
| `OR`            | Amplía los resultados incluyendo páginas con cualquiera de los términos. | `"linux" OR "ubuntu" OR "debian"`      | Busca páginas web que mencionen Linux, Ubuntu o Debian.                                                 |
| `NOT`           | Excluye resultados que contengan el término especificado.               | `site:bank.com NOT inurl:login`         | Encuentra páginas en bank.com excluyendo las de inicio de sesión.                                       |
| `*` (comodín)   | Representa cualquier carácter o palabra.                               | `site:socialnetwork.com filetype:pdf user* manual` | Busca manuales de usuario (guía de usuario, manual de usuario) en formato PDF en socialnetwork.com. |
| `..` (búsqueda de rango) | Encuentra resultados dentro de un rango numérico especificado. | `site:ecommerce.com "price" 100..500`   | Busca productos con precios entre 100 y 500 en un sitio de comercio electrónico.                       |
| `""` (comillas) | Busca frases exactas.                                                  | `"information security policy"`         | Encuentra documentos que mencionen la frase exacta "information security policy".                        |
| `-` (signo de resta) | Excluye términos de los resultados de búsqueda.                   | `site:news.com -inurl:sports`           | Busca artículos de noticias en news.com excluyendo contenido relacionado con deportes.

Una buena página para generar dorks es [pentest tools](https://pentest-tools.com/information-gathering/google-hacking).
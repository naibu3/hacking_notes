Joomscan es una herramienta de línea de comandos diseñada específicamente para escanear sitios web que utilizan [[Joomla]] y buscar posibles vulnerabilidades y debilidades de seguridad.

Joomscan utiliza una variedad de técnicas de enumeración para identificar información sobre el sitio web de Joomla, como la versión de Joomla utilizada, los plugins y módulos instalados y los usuarios registrados en el sitio. También utiliza una base de datos de vulnerabilidades conocidas para buscar posibles vulnerabilidades en la instalación de Joomla.

Para utilizarse debe descargarse del [repo oficial](https://github.com/OWASP/joomscan).

# Uso

```bash
perl joomscan.pl -u <url>
```
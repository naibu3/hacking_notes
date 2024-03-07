Drupal es un sistema de gestión de contenido libre y de código abierto ([[CMS]]) utilizado para la creación de sitios web y aplicaciones web.

Drupal ofrece un alto grado de personalización y escalabilidad, lo que lo convierte en una opción popular para sitios web complejos y grandes. Drupal se utiliza en una amplia gama de sitios web, desde blogs personales hasta sitios web gubernamentales y empresariales. Es altamente flexible y cuenta con una amplia variedad de módulos y herramientas que permiten a los usuarios personalizar su sitio web para satisfacer sus necesidades específicas.

# Reconocimiento

Como el resto de CMSs, lo suyo es comenzar con [[whatweb]] o [[wappalizer]].

Una herramienta útil es [[droopescan]]:

```bash
droopescan scan drupal --url <url>
```

# POC

Para la prueba de concepto utilizaremos el contenedor del siguiente [repo](https://github.com/vulhub/vulhub/tree/master/drupal/CVE-2018-7600).

Lanzaremos [[droopescan]]:

```bash
droopescan scan drupal --url http://127.0.0.1:8080
```
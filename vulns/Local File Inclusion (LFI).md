La vulnerabilidad **Local File Inclusion** (**LFI**) es una vulnerabilidad de seguridad informática que se produce cuando una aplicación web **no valida adecuadamente** las entradas de usuario, permitiendo a un atacante **acceder a archivos locales** en el servidor web.

En muchos casos, los atacantes aprovechan la vulnerabilidad de LFI al abusar de parámetros de entrada en la aplicación web. Los parámetros de entrada son datos que los usuarios ingresan en la aplicación web, como las URL o los campos de formulario. Los atacantes pueden manipular los parámetros de entrada para incluir rutas de archivo local en la solicitud, lo que puede permitirles acceder a archivos en el servidor web. Esta técnica se conoce como “**Path Traversal**” y se utiliza comúnmente en ataques de LFI.

En el ataque de Path Traversal, el atacante utiliza caracteres especiales y caracteres de escape en los parámetros de entrada para navegar a través de los directorios del servidor web y acceder a archivos en ubicaciones sensibles del sistema.

Por ejemplo, el atacante podría incluir **`../`** en el parámetro de entrada para navegar hacia arriba en la estructura del directorio y acceder a archivos en ubicaciones sensibles del sistema.

# Evasión de sanitizaciones

- ## Escapar de un directorio

Como hemos dicho antes, mediante `../` podemos salir del directorio en el que se encuentra la página. En caso de que haya alguna protección como **str_replace** de [[php]], podemos probar a poner `....//`.

- ## Escapar el uso de regexp

En caso de que mediante [[expresiones regulares]] se limite el acceso a un directorio o algún archivo, podemos incluir más barras o `./` a la ruta: `/etc////.//passwd`.

- ## Evitar que se añada una extensión de archivo

Cuando se concatena una extensión de archivo, podemos, en caso de existir una versión antigua de [[php]], un **nullbyte**: `\0` (url-encodeado: `%00`).

# Wrappers

Permiten encodear el output, algunos de los más comunes son:

- **php://filter/convert.base64-encode/resource=**
- **php://filter/filter/read=string.rot13/resource=**
- **php://filter/convert.iconv.utf-8.utf-16/resource=**
- **php://temp**

Mediante wrappers podemos llegar a ejecutar comandos, derivando un **LFI** en un [[RCE]]. Algunos son:

- **expect://\<comando\>
- **php://input**, permite, enviando la petición por *POST*, enviar datos que se ejecuten, por ejemplo: `<?php system['<command>']; ?>`
- **data://text/plain;base64,<comando en base64>**

En caso de que se estén aplicando codificaciones, podemos tratar de introducir código al comienzo de una cadena en base64. De forma que al decodificarse se ejecute. Para saber la codificación de cada carácter puedes consultar el siguiente [repo](https://github.com/synacktiv/php_filter_chain_generator).

# Log poisoning

A partir de un **LFI** podemos lograr [[RCE]] mediante un [[Log poisoning]].
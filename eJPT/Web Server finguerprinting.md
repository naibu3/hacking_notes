#banner_grabbing

El reconocimiento de un servidor web ó *web server finguerprinting* se refiere a la detección de información sobre el servidor  que corre el servicio [[80-http]], como puede ser:

- El **servicio web** como *nginx*, *Apache*, *IIS*...
- Su **versión**.
- El **sistema operativo** de la máquina host.


## Reconocimiento manual

Con el reconocimiento manual tenemos la opción de enviar los paquetes que queramos y aprovechar el banner para extraer información. Sin embargo, en ocasiones, el administrador del server puede modificar el banner para dificultar dicha práctica, en tal caso es recomendable optar por herramientas automatizadas.

### Netcat

Utilizaremos [[netcat]] como cliente para mandar peticiones al servidor. Esto se conoce como *banner grabbing*. Conectarnos aun servidor http con netcat es tan sencillo como:

```bash
nc <ip> 80
```

Una vez conectados debemos mandar una petición HTTP, lo cuál podemos hacerlo con `HEAD` (teniendo en cuenta que entre HEAD y el cuerpo hay dos líneas en blanco).

```http
HEAD / HTTP/1.0
```

Tras mandar otras dos líneas en blanco, el servidor debería responder. En la respuesta suele aparecer un campo `Server:`, que contiene información relevante sobre el propio servidor.

Ejemplo con un servidor http con python3:

```bash
# SERVER
sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
127.0.0.1 - - [16/Feb/2023 22:00:41] "HEAD / HTTP/1.0" 200 -
```
```bash
# NETCAT
nc localhost 80
HEAD / HTTP/1.0

HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.9.2
Date: Thu, 16 Feb 2023 21:00:41 GMT
Content-type: text/html; charset=utf-8
Content-Length: 3362
```

Sin embargo debemos tener en cuenta que netcat no realiza encriptación, por lo que no podemos conectarnos por http**s**.

### OpenSSL

Para conexiones por https utilizaremos [[OpenSSL]]. Al igual que antes debemos conectarnos:

```bash
openssl s_client -connect <ip>:<port>
```

Y mandar una petición:

```http
HEAD / HTTP/1.0
```


## Reconocimiento con herramientas automatizadas

### Httprint

[[httprint]] es una herramienta de reconocimiento con una técnica basada en la firma. La sintaxis más utilizada sería:

```bash
httprint -P0 -h <hosts> -s <signature-file>
```

- `-P0` evita hacer ping a la máquina (muchas no responden a las peticiones *ping echo*).
- `-h` especifica el ó los hosts.
- `-s` especifica el archivo de firma.
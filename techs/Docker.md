
Docker nos permite crear *contenedores*, es decir sistemas que son similares a las *máquinas virtuales* pero que comparten el kernel del *SO* nativo. De esta forma podemos instalar aplicaciones sin preocuparnos por las dependencias.

# Dockerfiles

Un *dockerfile* es un archivo que contiene la info necesaria para ejecutar un contenedor. Su estructura es:

```dockerfile
FROM ubuntu:latest

MAINTAINER Name Surname aka H4x0r "haxor@hack.com"

ENV DEBIAN_FRONTEND noninteractive

RUN apt update && apt install -y net-tools \
	iputils-ping \
	curl \
	git \
	nano \
	apache2 \
	php

ENTRYPOINT service apache2 start && /bin/bash
```
> Las `\` son para poder hacer saltos de linea.
> La variable de entorno `DEBIAN_FRONTEND noninteractive` hace que no entre en modo interactivo.

Algunas secciones comunes son:

- **FROM**: se utiliza para especificar la imagen base desde la cual se construirá la nueva imagen.
- **RUN**: se utiliza para ejecutar comandos en el interior del contenedor, como la instalación de paquetes o la configuración del entorno.
- **COPY**: se utiliza para copiar archivos desde el sistema host al interior del contenedor.
- **CMD**: se utiliza para especificar el comando que se ejecutará cuando se arranque el contenedor.

## Ejecutar un contenedor

Para ejecutar un contenedor desde un dockerfile `Dockerfile`, debemos utilizar el comando `docker build`:

```bash
docker build [opciones] ruta_al_Dockerfile
```
Con el parámetro `-t`, podemos especificar un nombre y una etiqueta:

```bash
docker build -t mi_imagen:v1 ruta_al_Dockerfile
```

Si en lugar de crear un *dockerfile* quisiéramos tomar la imagen desde un registro de archivos como [dockerhub](https://hub.docker.com/), debemos usar el comando `docker pull`:

```bash
docker pull nombre_de_la_imagen:etiqueta
```

Con `docker images`, podemos listar las imágenes que hay en uestro sistema y que podríamos utilizar a la hora de crear un contenedor:

```bash
docker images [opciones]
```

Para correr un contenedor utilizamos `docker run`:

```bash
docker run [opciones] nombre_de_la_imagen
```
```bash
docker run -dit mi_imagen
```

- `-d` o `--detach`: se utiliza para arrancar el contenedor en segundo plano, en lugar de en primer plano.
- `-i` o `-–interactive`: se utiliza para permitir la entrada interactiva al contenedor.
- `-t` o `–tty`: se utiliza para asignar un seudoterminal al contenedor.
- `-–name`: se utiliza para asignar un nombre al contenedor.

Con `docker ps` podemos ver los contenedores que están corriendo actualmente.

- `-a` o `-–all`: se utiliza para listar todos los contenedores, incluyendo los contenedores detenidos.
- `-q` o `-–quiet`: se utiliza para mostrar sólo los identificadores numéricos de los contenedores.

Para ejecutar comandos en un contenedor que ya está en ejecución, se utiliza el comando “**docker exec**” con diferentes opciones. Algunas de las opciones más comunes son:

```bash
docker exec -it <id> <comando>
```

- `-i` o `-–interactive`: se utiliza para permitir la entrada interactiva al contenedor.
- `-t` o `--tty`: se utiliza para asignar un seudoterminal al contenedor.

Para parar un contenedor se utiliza el comando `docker stop <id>` y para borrarlo `docker rm <id>`. Para borrar imágenes `docker rmi <id>`.

Para borrar todos los contenedores:

```bash
docker rm $(docker ps -a -q) --force
```

Borrar todas las imágenes:

```bash
docker rmi $(docker images -q)
```

# Port forwarding

Permite que los puertos de nuestra máquina estén conectados a los del contenedor:

```bash
docker run -p 80:8080 mi_imagen
```
```bash
docker run -p 53:53/udp mi_imagen
```

# Monturas

Las monturas nos permiten compartir un directorio de nuestra máquina en el contenedor:

```bash
docker run -v /home/usuario/datos:/datos mi_imagen
```

```bash
docker run -v /home/usuario/datos:/datos:ro mi_imagen
```
> Solo lectura (`ro`).

También se puede hacer mediante la sección **COPY** en el *Dockerfile*.

# Logs

Para ver los logs se utiliza `docker log <id>`.

# Docker-compose

Docker Compose es una herramienta de orquestación de contenedores que permite definir y ejecutar aplicaciones multi-contenedor de manera fácil y eficiente. Con Docker Compose, podemos describir los diferentes servicios que componen nuestra aplicación en un **archivo YAML** y, a continuación, utilizar un solo comando para ejecutar y gestionar todos estos servicios de manera coordinada.

Esto es especialmente útil para aplicaciones complejas que requieren la interacción de varios servicios diferentes, ya que Docker Compose permite definir y configurar fácilmente la conexión y la comunicación entre estos servicios.

En el [[GitHub]] de [vulhub](https://github.com/vulhub/vulhub), podremos encontrar gran cantidad de entornos vulnerables preconfigurados.

# Volúmenes

Mediante volúmenes, podemos guardar información que se utilizará la próxima vez que se cree un contenedor igual.

Para listarlos:

```bash
docker volume ls
```

Para borrarlos:

```bash
docker volume rm
```

Para borrar todos:

```bash
docker volume rm $(docker volume ls -q)
```

# Redes

Docker puede utilizarse para crear **redes personalizadas** en las que podremos simular un escenario de **red interna**. Para crear una nueva red en Docker, podemos utilizar el siguiente comando:

```bash
docker network create --subnet=<subnet> <nombre_de_red>
```
> **subnet**: es la dirección IP de la subred de la red que estamos creando. Es importante tener en cuenta que esta dirección IP debe ser única y no debe entrar en conflicto con otras redes o subredes existentes en nuestro sistema.
> **nombre_de_red**: es el nombre que le damos a la red que estamos creando.

Además de los campos mencionados anteriormente, también podemos utilizar la opción ‘**--driver**‘ en el comando ‘docker network create’ para especificar el controlador de red que deseamos utilizar.

Por ejemplo, si queremos crear una red de tipo “**bridge**“, podemos utilizar el siguiente comando:

```bash
docker network create --subnet=<subnet> --driver=bridge <nombre_de_red>
```

En este caso, estamos utilizando la opción ‘**--driver=bridge**‘ para indicar que deseamos crear una red de tipo “**bridge**“. La opción –driver nos permite especificar el controlador de red que deseamos utilizar, que puede ser “**bridge**“, “**overlay**“, “**macvlan**“, “**ipvlan**” u otro controlador compatible con Docker.
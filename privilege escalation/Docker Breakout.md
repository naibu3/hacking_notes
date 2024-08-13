#privesc 

# ¿Qué es?

Son técnicas para abusar de **[[Docker]]** con el objetivo de elevar nuestros privilegios de usuario y escapar del contenedor hacia la máquina host.

# Técnicas

- Uso de **monturas** en el despliegue de contenedores para acceder a archivos privilegiados del sistema host. Analizaremos cómo un atacante puede aprovechar las monturas para manipular los archivos del host y comprometer la seguridad del sistema.

- Despliegue de contenedores con la compartición de procesos (**–pid=host**) y permisos privilegiados (**–privileged**). Veremos cómo inyectar un shellcode ([[Injecting shellcode]]) malicioso en un proceso en ejecución como root, lo que podría permitir al atacante tomar control del sistema.

- Uso de **Portainer** para administrar el despliegue de un contenedor. Discutiremos cómo, mediante el empleo de monturas, un atacante podría ingresar y manipular archivos privilegiados del sistema host y escapar del contenedor.

- Abuso de la **API** de **Docker** por el puerto **2375** para la creación de imágenes, despliegue de contenedores e inyección de comandos privilegiados en la máquina host. Examinaremos cómo un atacante puede explotar la API de Docker para comprometer la seguridad del host y lograr la ejecución de comandos con privilegios elevados.
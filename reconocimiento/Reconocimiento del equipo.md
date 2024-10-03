Una vez ganas acceso a una máquina se debe aplicar un reconocimiento del equipo para encontrar vías de escalada de privilegios.

# Linux

## Herramientas

Para un primer reconocimiento puede ser interesante la herramienta [[lse]] ó [[linpeas]]. También se puede jugar con [[linenum]], aunque es un poco más viejo.

## Sudo

Con `sudo -l` podemos enumerar qué archivos podemos ejecutar como el usuario administrador.

## SUID

Podemos buscar permisos **suid** con:

```bash
find / -perm -4000 -ls 2>/dev/null
```
> Luego podemos buscar dichos binarios en [gtfobins]([GTFOBins](https://gtfobins.github.io/)).

## Capabilities

También podemos buscar **capabilities**, con [[getcap]].

## Crontabs

Otra opción es comprobar que tareas hay programadas con [[cron]] (`crontab -l`). O directamente leyendo el `/etc/crontab` ó el `/etc/timers`.

## Procesos

Para poder ver todos los procesos que corren podemos utilizar [[pspy]].

## Conexiones

Manualmente, podemos tratar de listar el `/proc/net/tcp` para ver *puertos abiertos en la máquina*:

```bash
cat /proc/net/tcp
```

Para *ver conexiones con otras máquinas*:

```bash
cat /proc/net/arp
```

Para ver si hay puertos *abiertos en esas otras máquinas*, podemos lanzarles una cadena vacía de la siguiente manera y ver el código de estado (sin es exitoso, *0*, están abiertos):

```bash
echo '' > /dev/tcp/<ip>/<port>
```
```bash
$?
```


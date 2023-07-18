Una vez ganas acceso a una máquina se debe aplicar un reconocimiento del equipo para encontrar vías de escalada de privilegios.

# Linux

Para un primer reconocimiento puede ser interesante la herramienta [[lse]]. También se puede jugar con [[linenum]], aunque es un poco más viejo.

Con `sudo -l` podemos enumerar qué archivos podemos ejecutar como el usuario administrador. Podemos buscar permisos **suid** con:

```bash
find / -perm -4000 -ls 2>/dev/null
```
> Luego podemos buscar dichos binarios en [gtfobins]([GTFOBins](https://gtfobins.github.io/)).

También podemos buscar **capabilities**, con [[getcap]].

Otra opción es comprobar que tareas hay programadas con [[cron]] (`crontab -l`). O directamente leyendo el `/etc/crontab` ó el `/etc/timers`.

Para poder ver todos los procesos que corren podemos utilizar [[pspy]].


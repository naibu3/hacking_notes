Es una herramienta para buscar hosts disponibles haciendo un barrido de todo el rango de direcciones de una red.

## Uso

```bash
fping -a -g <ip range>
```
```bash
fping -a -g 10.10.12.0/24
fping -a -g 10.10.12.0 10.10.12.255
```

Con el parámetro `-a`, mostramos sólo los hosts activos, y con `-g`, indicamos que queremos lanzar un barrido y no sólo un ping convencional. A veces incluso con el parámetro `-a` en LANs, aparecen mensajes de error, por lo que es recomendable enviarlos al `/dev/null` (`2>/dev/null`).
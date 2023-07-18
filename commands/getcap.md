Con el comando **getcap** podemos buscar *capabilities*, que nos permitan realizar ciertas acciones de forma privilegiada.

# Uso

Para buscar capabilities:

```bash
getcap -r / 2>/dev/null
```
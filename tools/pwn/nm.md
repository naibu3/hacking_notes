#tool #pwn 

# ¿Qué es?

Es una herramienta presente en las últimas versiones de Unix y sistemas operativos similares, se usa como ayuda en la depuración al examinar ficheros binarios, incluyendo bibliotecas, módulos objeto y ejecutables.

# Uso

## Ver símbolos de un binario

```bash
nm -a <file> | grep " t\| T"
```
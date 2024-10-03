#pwn 

# ¿Qué es?

**Ropper** es una herramienta de análisis de binarios. Es muy útil en ataques de tipo [[ROP - Return Oriented Programming|ROP]].

# Uso

## Buscar ROP gadgets

```bash
ropper -f <binary_name>
```

## Buscar strings

```bash
ropper -f <binary_name> --string <string>
```
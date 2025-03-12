# ¿Qué es?

*Alternate Data Streams* (**ADS**) es un atributo **NTFS** (*New Technology File System*) diseñado para lograr compatibilidad con el **HFS** (*Hierachical File System*) de MacOS.

Cualquier archivo creado en un disco con formato NTFS tendrá dos diferentes *streams*:

- *Data Stream* - Por defecto, contiene los datos.
- *Resource Stream* - Contiene la metadata.

Un atacante puede utilizar ADS para ocultar archivos maliciosos. Esto se logra ocultando la parte maliciosa en el *Resource Stream* de un archivo legítimo.

# Creación

Para crear un archivo con un contenido oculto:

```powershell
notepad test.txt:secret.txt
```

Para ejecutar:

```powershell
start test.txt:payload.exe
```
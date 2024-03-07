Es una herramienta para automatizar gran parte del proceso de enumeración de procesos en windows para la escalada de privilegios. Utilizado en [[Archetype HTB]].

## Descarga

Puedes descargarlo desde [aquí](https://github.com/carlospolop/PEASS-ng/releases/download/refs%2Fpull%2F260%2Fmerge/winPEASx64.exe).


## Utilización

Antes de nada debemos subir el binario a la máquina víctima y desde ella ejecutarlo:

```powershell
.\winPEASx64.exe
```

Una vez ejecutado se analizará gran cantidad de archivos dándonos por pantalla una lista de los potencialmente sensibles.
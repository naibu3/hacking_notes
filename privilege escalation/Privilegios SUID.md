#privesc 

# ¿Qué es?

Un privilegio **SUID** (**Set User ID**) es un permiso especial que se puede establecer en un archivo binario en sistemas Unix/Linux. Este permiso le da al usuario que ejecuta el archivo los **mismos privilegios** que el **propietario** del archivo.

Por ejemplo, si un archivo binario tiene establecido el permiso SUID y es propiedad del usuario root, cualquier usuario que lo ejecute adquirirá temporalmente los mismos privilegios que el usuario root, lo que le permitirá realizar acciones que normalmente no podría hacer como un usuario normal.

# Buscarlos

Para buscar binarios con estos permisos podemos utilizar [[find]]:

```bash
find / -perm 4000 2>/dev/null
```
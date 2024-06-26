***Relocation Read-Only*** (ó **RELRO**), es una protección que hace ciertas secciones de un binario de **sólo-lectura**. Hay dos modos *Partial* y *Full*.

# Partial RELRO

Es el modo por defecto de GCC. Desde el punto de vista de un atacante no hace ninguna diferencia, excepto prevenir que variables globales sobrescriban la [[GOT]].

# Full RELRO

Hace que la [[GOT]] sea de sólo lectura, evitando ataques. Sin embargo hace que la carga sea mucho más lenta ya que las direcciones de las funciones utilizadas se deben resolver al inicio del programa, haciéndolo inviable en programas grandes.

No previene que se sobrescriban los valores de la [[GOT]] en caso de [[format string]].
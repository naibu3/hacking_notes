#web 

# ¿Qué es?

**XPath** es un lenguaje de consultas utilizado en **[[XML]]** que permite buscar y recuperar información específica de **documentos XML**.

# Tipos

Algunos tipos de vulnerabilidades comunes en XPath son:

- **Inyección XPath**: los atacantes pueden utilizar inyección de código malicioso en las consultas XPath para alterar el comportamiento esperado de la aplicación. Por ejemplo, pueden agregar una consulta maliciosa que recupere toda la información del usuario, incluso información confidencial como contraseñas.
- **Fuerza bruta de XPath**: los atacantes pueden utilizar técnicas de fuerza bruta para adivinar las rutas de XPath y recuperar información confidencial. Esta técnica se basa en intentar diferentes rutas XPath hasta encontrar una que devuelva información confidencial.
- **Recuperación de información del servidor**: los atacantes pueden utilizar consultas XPath maliciosas para obtener información sobre el servidor, como el tipo de base de datos, la versión de la aplicación, etc. Esta información puede ayudar a los atacantes a planear ataques más sofisticados.
- **Manipulación de respuestas XPath**: los atacantes pueden manipular las respuestas XPath de la aplicación web para obtener información adicional o alterar el comportamiento de la aplicación. Por ejemplo, pueden modificar una respuesta XPath para crear una cuenta de usuario sin permiso.

# Explotación

Supongamos que la consulta que realiza el servidor tiene este aspecto:

```XPath
/Coffees/Coffee[@ID='']
```

SI añadiéramos `1' or count(/*)='1` podemos tratar de contar el **número de elementos** bajo una etiqueta. Podemos ir probando con 2,3,4... Luego podemos intentar probar con las subetiquetas (aún sin saber su nombre), `1' or count(/*[1])=1'`.

Para los **nombres**, `1' or substring(name(/*[0]), 1, 1)='a` e iremos iterando los caracteres. Para la longitud podemos probar con `1' or string.length(name(/*[0]), 1, 1)='1`
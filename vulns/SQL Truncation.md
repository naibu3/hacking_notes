#vuln 

# ¿Qué es?

El ataque de **truncado SQL**, también conocido como **SQL Truncation**, es una técnica de ataque en la que un atacante intenta **truncar** o **cortar** una consulta SQL para realizar acciones maliciosas en una **base de datos**.

Suponiendo que por ejemplo el usuario `admin@admin.com` ya existe en la base de datos, de primeras no sería posible registrar este mismo usuario, debido a que la query SQL que se aplicaría por detrás lo consideraría como una entrada duplicada. Sin embargo, dado que el correo `admin@admin.com` tiene un total de **15 caracteres** y todavía no hemos llegado al límite, un atacante lo que podría intentar hacer es registrar al usuario `admin@admin.com  a` o de otra forma para que lo veáis mejor:`admin@admin.com[espacio][espacio]a`.

Esta nueva cadena que hemos representado para el correo electrónico, en este caso tiene un total de **18 caracteres**. De primeras el correo es distinto al correo ya existente en la base de datos (`admin@admin.com`), sin embargo, debido a su limitación en **17 caracteres**, tras pasar el primer filtro y proceder a su inserción en la base de datos, la longitud total de la cadena se acota a los 17 caracteres, resultando en la cadena `admin@admin.com` , o dicho de otra forma: `admin@admin.com[espacio][espacio]`.

Ahora bien, ¿qué pasa con los espacios?, pues dado que no representan “información de valor”, por decirlo de alguna forma, lo que sucederá es que se **truncarán**. A lo que nos referimos con esto del **truncado de los espacios** al final de la cadena, es a la eliminación automática de los mismos. De esta forma, la cadena resultante final se quedaría en `admin@admin.com`, consiguiendo tras su inserción en la base de datos cambiar su contraseña a la especificada durante la fase de registro para nuestro supuesto “**nuevo usuario**“.
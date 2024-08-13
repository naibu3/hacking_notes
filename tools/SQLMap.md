#tool 
# ¿Qué es?

**SQLMap** es una herramienta de pruebas de penetración de código abierto que se utiliza para detectar y explotar vulnerabilidades de inyección SQL en aplicaciones web. Esta herramienta se basa en un motor de inyección SQL altamente automatizado que puede detectar y explotar una amplia variedad de vulnerabilidades de inyección SQL en diferentes sistemas de gestión de bases de datos.

# Uso

## Enumerar bases de datos

```bash
sqlmap -u <url_vulnerable> --dbs
```

## Enumerar tablas

```bash
sqlmap -u <url_vulnerable> --dbms mysql --batch -D <database> --tables
```

## Enumerar columnas

```bash
sqlmap -u <url_vulnerable> --dbms mysql --batch -D <database> -T <table> --columns
```
```bash
sqlmap -u <url_vulnerable> --dbms mysql --batch -D <database> -T <table> --C <column1,column2> --dump
```

## Especificar cookies

```bash
sqlmap -u <url_vulnerable> --cookie "PHPSESSID=sahdksajhdjksahdsajkdhsakjdh"
```

## Especificar tipo de DB

```bash
sqlmap -u <url_vulnerable> --dbms mysql
```

## Default

Para que no se nos pregunte acerca de opciones específicas, podemos utilizar la opción **`--batch`**.

## Risk/level

Con **`--risk`** y **`--level`** podemos especificar la cantidad de ataques que probará la herramienta.

## Shell

Con el parámetro **`--os-shell`** podemos tratar de obtener una shell en caso de que sea posible.
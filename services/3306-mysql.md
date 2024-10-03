---
title: 3306-mysql
author: naibu3
---
# ¿Qué es?

*MySQL* es un servicio de base de datos relacional que suele correr por el puerto *`3306`*. Utiliza la estructura *cliente-servidor*. Para manejar los datos hace uso del lenguaje SQL.

Es un servicio de base de datos ampliamente utilizado, tanto por [[CMS]] como [[Wordpress]] como en stacks de tecnología como [LAMP](https://en.wikipedia.org/wiki/LAMP_(software_bundle)) ó [LEMP](https://lemp.io/). Conteniendo normalmente información sensible. Al formar parte de otras tecnologías, en ocasiones la mala implementación puede llevar a errores como [[SQL Injection]].

## MariaDB

Es un *fork* de MySQL iniciado cuando el jefe de desarrollo de MySQL abandonó la empresa tras ser comprada por Oracle.

# Configuración

La configuración de un servicio de base de datos es algo complejo y responsabilidad del `database administrator`. Podemos ver la configuración de la siguiente forma:

```bash
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
```

## Configuraciones peligrosas

|**Settings**|**Description**|
|---|---|
|`user`|Sets which user the MySQL service will run as.|
|`password`|Sets the password for the MySQL user.|
|`admin_address`|The IP address on which to listen for TCP/IP connections on the administrative network interface.|
|`debug`|This variable indicates the current debugging settings|
|`sql_warnings`|This variable controls whether single-row INSERT statements produce an information string if warnings occur.|
|`secure_file_priv`|This variable is used to limit the effect of data import and export operations.|

# Auditando el servicio

## Nmap 

Podemos hacer un primer reconocimiento con [[nmap]] utilizando los scripts de MySQL: `--script mysql*`.

## Conexión

```bash
mysql -u <user> -D <database> -h <host> -p <password>
```

Dos de las bases de datos más relevantes serán `system schema` (`sys`) e`information schema` (`information_schema`), ya que contienen información sobre el propio sistema y sobre el resto de bases de datos.

Algunos comandos útiles son:

|**Command**|**Description**|
|---|---|
|`mysql -u <user> -p<password> -h <IP address>`|Connect to the MySQL server. There should **not** be a space between the '-p' flag, and the password.|
|`show databases;`|Show all databases.|
|`use <database>;`|Select one of the existing databases.|
|`show tables;`|Show all available tables in the selected database.|
|`show columns from <table>;`|Show all columns in the selected database.|
|`select * from <table>;`|Show everything in the desired table.|
|`select * from <table> where <column> = "<string>";`|Search for needed `string` in the desired table.|



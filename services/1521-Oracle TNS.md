---
title: Oracle TNS
service: TCP
---
# ¿Qué es?

`Oracle Transparent Network Substrate` (`TNS`) es un protocolo que facilita la comunicación entre bases de datos de Oracle y aplicaciones. Por defecto, suele correr sobre el puerto `1521/TCP`.
# Configuración

La configuración depende de la versión del software de Oracle, aunque existen algunas configuraciones comunes. La configuración suele encontrarse en `tnsnames.ora` y `listener.ora` que suelen estar en el directorio `$ORACLE_HOME/network/admin`.

Suele configurarse para trabajar con otras aplicaciones de Oracle como Oracle DBSNMP, Oracle Databases, Oracle Application Server, Oracle Enterprise Manager y Oracle Fusion Middleware.

El servicio puede protegerse utilizando *PL/SQL Exclusion List* (`PlsqlExclusionList`). Son ficheros de texto creados por el usuario en `$ORACLE_HOME/sqldeveloper`, actuando como una *blacklist*:

|**Setting**|**Description**|
|---|---|
|`DESCRIPTION`|A descriptor that provides a name for the database and its connection type.|
|`ADDRESS`|The network address of the database, which includes the hostname and port number.|
|`PROTOCOL`|The network protocol used for communication with the server|
|`PORT`|The port number used for communication with the server|
|`CONNECT_DATA`|Specifies the attributes of the connection, such as the service name or SID, protocol, and database instance identifier.|
|`INSTANCE_NAME`|The name of the database instance the client wants to connect.|
|`SERVICE_NAME`|The name of the service that the client wants to connect to.|
|`SERVER`|The type of server used for the database connection, such as dedicated or shared.|
|`USER`|The username used to authenticate with the database server.|
|`PASSWORD`|The password used to authenticate with the database server.|
|`SECURITY`|The type of security for the connection.|
|`VALIDATE_CERT`|Whether to validate the certificate using SSL/TLS.|
|`SSL_VERSION`|The version of SSL/TLS to use for the connection.|
|`CONNECT_TIMEOUT`|The time limit in seconds for the client to establish a connection to the database.|
|`RECEIVE_TIMEOUT`|The time limit in seconds for the client to receive a response from the database.|
|`SEND_TIMEOUT`|The time limit in seconds for the client to send a request to the database.|
|`SQLNET.EXPIRE_TIME`|The time limit in seconds for the client to detect a connection has failed.|
|`TRACE_LEVEL`|The level of tracing for the database connection.|
|`TRACE_DIRECTORY`|The directory where the trace files are stored.|
|`TRACE_FILE_NAME`|The name of the trace file.|
|`LOG_FILE`|The file where the log information is stored.|



# Auditando el servicio

## Nmap

Como siempre, podemos utilizar [[nmap]]. Uno de los usos más importantes es para obtener un *SID* que nos permita autenticarnos en el servicio.

```bash
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute
```

## [[ODAT]]

```bash
./odat.py all -s 10.129.204.235
```

## Autenticarse en el servicio

Podemos conectarnos utilizando [[sqlplus]]:

```bash
sqlplus scott/tiger@10.129.204.235/XE
```

```shell-session
select table_name from all_tables;
```

```shell-session
sqlplus scott/tiger@10.129.204.235/XE as sysdba
```

```shell-session
select name, password from sys.user$;
```
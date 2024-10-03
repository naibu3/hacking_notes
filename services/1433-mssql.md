# ¿Qué es?

Es un servicio de base de datos creado por *Microsoft* para correr en servidores Windows, por lo que es software propietario. Suele correr en el puerto `1433/TCP`. Este servicio se explota en [[Archetype HTB]].

Para más info consultar:

- [HACKTRICKS](https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server)
- [PENTEST MONKEY](https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)

## Clientes MSSQL

[SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) (`SSMS`) es un sistema que permite gestionar el servicio y suele instalarse en el propio servidor para la configuración inicial. Sin embargo, es posible conectarse al servicio desde una gran cantidad de clientes. Para un atacante la aplicación más interesante será *mssqlclient.py* de [[impacket]].

## Bases de datos

|Default System Database|Description|
|---|---|
|`master`|Tracks all system information for an SQL server instance|
|`model`|Template database that acts as a structure for every new database created. Any setting changed in the model database will be reflected in any new database created after changes to the model database|
|`msdb`|The SQL Server Agent uses this database to schedule jobs & alerts|
|`tempdb`|Stores temporary objects|
|`resource`|Read-only database containing system objects included with SQL server|

# Configuración

Al instalarse, el servicio correrá como `NT SERVICE\MSSQLSERVER`, aunque se puede configurar como `Windows Authentication`, para que en caso de inicio de sesión sea el módulo SAM o el DC los que manejen la autenticación.

## Configuraciones peligrosas

- Los clientes no utilizan encriptación al conectarse al servicio.
- El uso de certificados autofirmados en caso de utilizar encriptación.
- El uso de [named pipes](https://docs.microsoft.com/en-us/sql/tools/configuration-manager/named-pipes-properties?view=sql-server-ver15).
- Credenciales por defecto para la cuenta de administrador.

# Auditando el servicio

## Nmap

Con [[nmap]] tenemos disponibles una serie de scripts que nos permitirán un escaneo de posibles vulnerabilidades del servicio `--script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER`

## Metasploit

También podemos utilizar un escáner de [[metasploit]] llamado `mssql_ping`:

## Conexión

Una vez tengamos credenciales podemos conectarnos utilizando T-SQL (`Transact-SQL`). Para la conexión desde Linux, utilizaremos el script de [[impacket]], llamado `mssqlclient.py`:

```bash
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth
```

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
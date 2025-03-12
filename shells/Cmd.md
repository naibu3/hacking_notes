*Cmd* es la terminal de windows. Se suele encontrar en `C:\Windows\system32\cmd.exe`. Al contrario que [[Bash]], posee varias funcionalidades integradas (bash llama a biarios externos). Actualmente windows está desarrollando más la otra opción de terminal, [[techs/Powershell]].

# 1. Windows environment

Al igual que [[Bash]], windows tiene su propio entorno, aunque suele interactuarse con una interfaz gráfica. Para interactuar con las **variables de entorno**, debes ir a `Panel de Control > System and Security > System > Advanced System Settings` y administrarlas globalmente ó para cada usuario.

Una de las variables de entorno más importantes es el **`PATH`**, al igual que en bash, pero separada con `;`. Cuando cmd.exe va a ejecutar un programa primero comprueba si es uno de los comando integrados (*built-in*), y en caso de no serlo lo busca en el `PATH`.

En windows en lugar de `/` (*slash*), se utiliza `\` (*backslash*).

Las posibilidades de scripting en cmd son más limitadas que en bash, ya que los scripts se escribirán en powershell. Podemos crear scripts en archivos `.bat`, con una instrucción por linea.

### 1.1 Variables

Para acceder a **variables de entorno** se hará con la siguiente sintaxis:

```powershell
echo %PATH%
echo %username%
```

Para ver otras variables utilizaremos `set`:

```powershell
set
```

Además podemos crear o modificar variables, pero sólo afectará a la pestaña actual.


## 2. Encadenar comandos

Podemos encadenar comandos con:

```powershell
echo hola & echo adios #Encadena ambos sin importar el resultado
echo hola && echo quetal #Encadena ambos solo si el primero tiene éxito
echo hola || acho adios #Ejecuta el segundo solo si el primero no tiene éxito
```

Igualmente los operadores de redirección también funcionan:

```powershell
echo hola > file.txt
echo quetal >> file.txt
```

Podemos ver los archivos con `type`.

También podemos pasar la salida de un comando como parámetro al siguiente con un *pipe*, al igual que en bash:

```powershell
comand1 | comand2
```

Para que un comando muestre solo la salida, utilizaremos `@`, por ejemplo `@echo hola`.

## 3. Bucles y condicionales

### 3.1 IF

```powershell
if <condition> (<action>) else (<action>)
```

Para más info mira aquí: [if](https://ss64.com/nt/if.html) y [else](https://ss64.com/nt/else.html).

### 3.2 FOR

```powershell
for %i in <site> do <action>
```

```powershell
#Lista los archivos en un directorio
for %i in (*.*) do @echo FILE:%i
```

Para más info mira [aquí](https://ss64.com/nt/for.html).
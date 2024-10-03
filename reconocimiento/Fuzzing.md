En ocasiones, los usuarios o motores de búsqueda no pueden encontrar recursos que no estén linkados a una página en internet. Sin embargo, estos recursos siguen siedo accesibles mediante una *URL*. Mediante técnicas de enumeración podemos encontrar estos recursos.

Hay dos opciones a la hora de enumerar recursos, la primera es utilizar fuerza bruta, lo que es normalmente muy ineficiente. La segunda consiste en utilizar diccionarios con nombres comunes. Para ello existen varias herramientas que automatizan el proceso, tales como [[dirbuster]], ...

# Metodología

1. En primer lugar, buscar *directorios*.
2. Además buscar archivos mediante extensiones (`.php`, ...)
3. Enumerar subdominios ([[Subdomain enumeration]])
# Herramientas
## [[dirbuster]]

Es una herramienta desarrollada en java por OWASP que aunque ya no tiene soporte sigue siendo muy utilizada.

## [[gobuster]]

[[gobuster]] es una herramienta de *fuzzing* escrita en *go*. Para enumeración de recursos utilizaremos el modo `dir`:

```bash
gobuster dir -u <url> -w <wordlist> -t 200
```
> `-t` es para especificar el número de hilos.
> En ocasiones es necesario el `--add-slash`, para que añada una `/` a las direcciones.
> Con `-b` podemos ocultar ciertos códigos de estado.
> Si quisiéramos buscar archivos, con `-x` podemos *fuzzear* añadiendo una cierta extensión.


## [[wfuzz]]

[[wfuzz]] es otra muy buena herramienta de *fuzzing*.

```bash
wfuzz -c --hc=404,403 -t 200 -w <wordlist> http://dominio/FUZZ/
```
> `-c` es para que se muestre la salida con colores.
> `-t` es para especificar el número de hilos.
> `--hc` es para ocultar cierto código de estado.
> `-L` hace que siga las redirecciones (aunque es mejor añadir una barra directamente).
> `--sl` permite mostrar respuestas con un número específico de líneas.


## [[ffuf]]

[[ffuf]] es otra herramienta de *fuzzing* escrita en *go* por lo que trabaja bien con conexiones.

## BurpSuite

Se puede hacer *fuzzing* con [[burpsuite]], mediante la sección de *sitemap*.

# Enumeración pasiva

Podemos hacer enumeración pasiva con webs como [Phonebook](https://phonebook.cz/).

# Wordlists

Normalmente se suele emplear el `/usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt`.
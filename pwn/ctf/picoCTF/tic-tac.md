#ctf #pwn 

# Reconocimiento

Nos dan una conexión por [[22-ssh]] a un servidor que contiene un binario que aparentemente permite listar el contenido de un fichero en caso de que nos pertenezca. Además tenemos el código fuente:

```c
#include <iostream>
#include <fstream>
#include <unistd.h>
#include <sys/stat.h>

int main(int argc, char *argv[]) {
  if (argc != 2) {
    std::cerr << "Usage: " << argv[0] << " <filename>" << std::endl;
    return 1;
  }

  std::string filename = argv[1];
  std::ifstream file(filename);
  struct stat statbuf;

  // Check the file's status information.
  if (stat(filename.c_str(), &statbuf) == -1) {
    std::cerr << "Error: Could not retrieve file information" << std::endl;
    return 1;
  }

  // Check the file's owner.
  if (statbuf.st_uid != getuid()) {
    std::cerr << "Error: you don't own this file" << std::endl;
    return 1;
  }

  // Read the contents of the file.
  if (file.is_open()) {
    std::string line;
    while (getline(file, line)) {
      std::cout << line << std::endl;
    }
  } else {
    std::cerr << "Error: Could not open file" << std::endl;
    return 1;
  }

  return 0;
}
```

# Explotación

## Explicación

De primeras podemos tratar de crear un *enlace simbólico* para que éste si que nos pertenezca, pero el programa utiliza la función **`stat()`** que sigue los enlaces hasta el original.

Si nos fijamos en los *tags* del propio reto, veremos uno que dice **toctou**, que significa *time-of-check time-of-use* y hace referencia a una [[Race Conditions]]. Que aparece porque primero comprueba si el usuario posee el fichero y luego lo lee. Estas dos acciones, durante milisegundos se dan a la vez, permitiéndonos leer el archivo.

## Explotación de la race condition

Lo que debemos hacer es crear un *enlace simbólico* a un archivo que poseemos y rápidamente cambiar dicho enlace para que apunte al archivo que queremos leer. Podemos hacerlo mediante un *one-liner* en [[Bash]]:

```bash
timeout 30s bash -c 'while true; do ln -sf src.cpp flag; ln -sf flag.txt flag; done' &
```
> El `&` ejecuta el comando en segundo plano, permitiéndonos ejecutar la segunda parte.

La segunda parte será otro *one-liner*:

```bash
while ! ./txtreader flag 2> /dev/null | grep "picoCTF"; do :; done
```

Solo nos queda ejecutarlo y después de un tiempo, ocurrirá que se leerá el fichero con la flag.
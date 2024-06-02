# fd - pwnable.kr

## Analisis del código

El servidor contiene dos archivos, un binario y lo que asumimos que es su código fuente.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
    if(argc<2){
        printf("pass argv[1] a number\n");
        return 0;
    }
    int fd = atoi( argv[1] ) - 0x1234;
    int len = 0;
    len = read(fd, buf, 32);
    if(!strcmp("LETMEWIN\n", buf)){
        printf("good job :)\n");
        system("/bin/cat flag");
        exit(0);
    }
    printf("learn about Linux file IO\n");
    return 0;

}
```

Vemos que el programa recibe un argumento al que le resta 0x1234 (4660 en decimal) y guarda el resultado en **fd** (descriptor de fichero).
Seguidamente, lee el archivo descrito por fd y lo almacena en **buff**. Finalmente, comprueba si buff es igual a la cadena *LETMEWIN* y,
en caso de serlo, nos muestra la flag.

## Explotación

Si buscamos en el manual, podemos ver que la función **read** puede generar errores en caso de recibir un descriptor igual a cero.

Por tanto, podemos tratar de poner dicho descriptor a cero, introduciendo como parámetro 4660 (0x1234 - 0x1234 = 0). Si probamos,
se nos quedará el programa en la función *read* a la espera de que introduzcamos un valor manualmente, introducimos *LETMEWIN* y
vemos que nos reporta la flag.

```fd
fd@pwnable:~$ ./fd 4660
LETMEWIN
good job :)
mommy! I think I know what a file descriptor is!!
```

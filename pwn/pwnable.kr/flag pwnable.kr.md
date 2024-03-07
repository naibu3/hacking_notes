# flag - pwnable.kr

## Análisis

Se nos da simplemente un binario. Si comprobamos sus propiedades con *file*:

```bash
file flag
```
```file
flag.original: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, no section header
```

Vemos que no nos dice *not stripped*, por lo que si probamos a debuguear con gdb no podremos, ya que se han eliminado
la información necesaria paara ello. Sin embargo, mediante *strings* podemos sacar algo más de información:

```bash
strings flag
```
```strings
Upbrk
makBN
UPX!
UPX!
```

Vemos UPX, un programa para empaquetar programas, si filtramos por *packed*:

```bash
strings flag.original | grep packed
```
```
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
```

Si investigamos, veremos que con el comando *upx* podemos desempaquetar el binario:

```bash
upx -d flag
```

Ya podemos debuguear con gdb:

```bash
gdb flag
```

```gdb
disass main
```
```
Dump of assembler code for function main:
   0x0000000000401164 <+0>:	push   rbp
   0x0000000000401165 <+1>:	mov    rbp,rsp
   0x0000000000401168 <+4>:	sub    rsp,0x10
   0x000000000040116c <+8>:	mov    edi,0x496658
   0x0000000000401171 <+13>:	call   0x402080 <puts>
   0x0000000000401176 <+18>:	mov    edi,0x64
   0x000000000040117b <+23>:	call   0x4099d0 <malloc>
   0x0000000000401180 <+28>:	mov    QWORD PTR [rbp-0x8],rax
   0x0000000000401184 <+32>:	mov    rdx,QWORD PTR [rip+0x2c0ee5]        # 0x6c2070 <flag>
   0x000000000040118b <+39>:	mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040118f <+43>:	mov    rsi,rdx
   0x0000000000401192 <+46>:	mov    rdi,rax
   0x0000000000401195 <+49>:	call   0x400320
   0x000000000040119a <+54>:	mov    eax,0x0
   0x000000000040119f <+59>:	leave  
   0x00000000004011a0 <+60>:	ret
```

Vemos que nos dicen la posición que almacena la flag, así que la imprimimos:

```gdb
x/1s 0x6c2070
```
```
0x496628:	"UPX...? sounds like a delivery service :)"
```

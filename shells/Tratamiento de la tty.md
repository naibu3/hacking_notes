A veces, al obtener una reverse shell, no tenemos una shell completamente interactiva. Para conseguirla, debemos aplicar un tratamiento de la tty.

# Método 1

Comenzamos ejecutando:

```shell
script /dev/null -c bash
```

Una vez ejecutado pulsamos CTL+Z para mandar el proceso a segundo plano, y ejecutamos:

```shell
stty raw -echo;fg
```

Y se nos quedará la teminal detenida, simplemente introducimos:

```shell
reset
```
```bash
xterm
```

Y en este punto ya tendríamos una shell completamente interactiva (historial, navegación con las flechas, CTL+C, ...). Aunque en caso de error, sería recomendable ejecutar:

```shell
export TERM=xterm; export SHELL=bash
```

Para ajustar la resolución:

```bash
stty rows 54 columns 190
```

# Método 2

Este método solo difiere el el primer comando, que utiliza python:

```shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

Después procedería con CTL+Z y con las siguientes líneas de igual forma.
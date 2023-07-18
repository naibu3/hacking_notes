Es una herramienta de *fuzzing* escrita en *go* por lo que trabaja bien con conexiones.

## Uso

```bash
ffuf -c -t 200 -w <wordlist> -u http://domain/FUZZ/ --mc=200
```

`--mc` nos permite filtrar por un código en específico.
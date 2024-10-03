# ¿Qué es?

Es un editor de código para línea de comando.

# Uso 
## Modo normal

Por defecto, estaremos en *modo normal*. En este modo podemos:

| Command | Description    |
| ------- | -------------- |
| `x`     | Cut character  |
| `dw`    | Cut word       |
| `dd`    | Cut full line  |
| `yw`    | Copy word      |
| `yy`    | Copy full line |
| `p`     | Paste          |
A cualquiera de estos comandos, podemos añadirle un número para especificar la cantidad (por ejemplo, `3x` cortará 3 carácteres).
## Modo insertar

Con la `i`, entraremos en el modo insertar, y con `esc`, podemos salir.

## Modo comando

Con `:` entraremos en el modo comando, aquí podremos:

| Command | Description          |
| ------- | -------------------- |
| `:1`    | Go to line number 1. |
| `:w`    | Write the file, save |
| `:q`    | Quit                 |
| `:q!`   | Quit without saving  |
| `:wq`   | Write and quit       |
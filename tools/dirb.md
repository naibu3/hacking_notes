Es la alternativa para la terminal de linux a [[dirbuster]].

## Instalación

```bash
sudo apt install dirb
```

## Uso

Simplemente debemos especificar el target, y en caso de querer también el wordlist (trae uno por defecto):

```bash
dirb <target> [wordlist]
```

#### Redirigir el tráfico a través de BurpSuite

Para que el tráfico se redirija a través del proxy de [[burpsuite]], debemos utilizar el parámetro `-p` (proxy):

```bash
dirb <target> -p http://127.0.0.1:8080
```

#### Delay

En caso de que no podamos mandar las peticiones muy rápido, podemos aplicar un delay entre peticiones con `-z <delay_en_ms>`.

#### Silent mode

Para evitar que nos salga demasiada basura en la pantalla podemos usar `-S` (silent mode).

#### Extensions

Podemos concatenar extensiones a un diccionario con `-X ".php"`. Ó desde un archivo con `-x archivoextens.txt`.
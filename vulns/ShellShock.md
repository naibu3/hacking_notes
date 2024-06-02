# ¿Qué es?

Un ataque **Shellshock** es un tipo de ataque informático que aprovecha una vulnerabilidad en el **intérprete de comandos Bash** en sistemas operativos basados en Unix y Linux. Esta vulnerabilidad se descubrió en 2014 y se considera uno de los ataques más grandes y generalizados en la historia de la informática.

# Indicadores

La existencia de una ruta ***/cgi-bin/***.

# Explotación

Los atacantes pueden explotar esta vulnerabilidad a través de diferentes vectores de ataque. Uno de ellos es a través del **User-Agent**, que es la información que el navegador web envía al servidor web para identificar el tipo de navegador y sistema operativo que se está utilizando. Los atacantes pueden manipular el User-Agent para incluir comandos maliciosos, que el servidor web ejecutará al recibir la solicitud.

Un ejemplo con [[curl]] sería:

```bash
curl -H "User-Agent: () { :; }; <command>" http://example.com
```

A veces hay que meter un `echo; ` antes del comando.

# Más info

Más info en este [artículo](https://blog.cloudflare.com/inside-shellshock).
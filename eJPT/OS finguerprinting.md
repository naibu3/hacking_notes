Una vez sabemos qué hosts están activos dentro de la red, podemos tratar de comprobar si se tratan de routers, clientes, servidores... Para ello podemos mandar ciertas peticiones y analizar la respuesta. Esto se debe a que cada sistema operativo tiene distintas implementaciones de pila. Así conseguiremos una tabla con todos los hosts, su SO y la confianza (porcentaje de acierto).

El OS finguerprinting puede realizarse online u offline. Por ahora nos concentraremos en la parte online con [[nmap]], aunque también podemos realizarlo mediante un script.

## Detección de SO mediante el ttl

A través de uno de los parámetros que nos reporta un [[ping]], el ***ttl***, podemos presuponer cuál puede ser el SO. Esta información está disponible en internet, pero nos basta con saber que un ttl igual a 64 corresponde a un sistema linux y un ttl de 128 a uno windows. En ocasiones, debido a la existencia de nodos intermedios este valor puede ser mayor o menor, sin embargo normalmente estará cercano a los valores especificados.

## Mediante nmap

Como hemos mencionado, es posible llevarlo a cabo con el parámetro `-O` en [[nmap]].
Una vez sabemos qué hosts están activos dentro de la red, podemos tratar de comprobar si se tratan de routers, clientes, servidores... Para ello podemos mandar ciertas peticiones y analizar la respuesta. Esto se debe a que cada sistema operativo tiene distintas implementaciones de pila. Así conseguiremos una tabla con todos los hosts, su SO y la confianza (porcentaje de acierto).

El OS finguerprinting puede realizarse online u offline. Por ahora nos concentraremos en la parte online con [[nmap]], aunque también podemos realizarlo mediante un script.

## Detección de SO mediante el ttl

El tiempo de vida (**TTL**) hace referencia a la cantidad de tiempo o “**saltos**” que se ha establecido que un paquete debe existir dentro de una red antes de ser descartado por un enrutador. El TTL también se utiliza en otros contextos, como el almacenamiento en caché de CDN y el almacenamiento en caché de DNS.

Cuando se crea un paquete de información y se envía a través de Internet, está el riesgo de que siga pasando de enrutador a enrutador indefinidamente. Para mitigar esta posibilidad, los paquetes se diseñan con una caducidad denominada **tiempo de vida** o **límite de saltos**. El TTL de los paquetes también puede ser útil para determinar cuánto tiempo ha estado en circulación un paquete determinado, y permite que el remitente pueda recibir información sobre la trayectoria de un paquete a través de Internet.

Cada paquete tiene un lugar en el que se almacena un valor numérico que determina cuánto tiempo debe seguir moviéndose por la red. Cada vez que un enrutador recibe un paquete, resta uno al recuento de TTL y lo pasa al siguiente lugar de la red. Si en algún momento el recuento de TTL llega a cero después de la resta, el enrutador descartará el paquete y enviará un mensaje ICMP al host de origen.

¿Qué tiene que ver esto con la identificación del sistema operativo? Bueno, resulta que diferentes sistemas operativos tienen diferentes valores predeterminados de TTL. Por ejemplo, en sistemas operativos Windows, el valor predeterminado de TTL es 128, mientras que en sistemas operativos Linux es 64. Podemos ver este valor mediante un simple [[ping]].

Utilidad de S4vitar (whichSystem):

```python
#!/usr/bin/python3
#coding: utf-8

import re, sys, subprocess

# python3 wichSystem.py 10.10.10.188 

if len(sys.argv) != 2:
    print("\n[!] Uso: python3 " + sys.argv[0] + " <direccion-ip>\n")
    sys.exit(1)

def get_ttl(ip_address):

    proc = subprocess.Popen(["/usr/bin/ping -c 1 %s" % ip_address, ""], stdout=subprocess.PIPE, shell=True)
    (out,err) = proc.communicate()

    out = out.split()
    out = out[12].decode('utf-8')

    ttl_value = re.findall(r"\d{1,3}", out)[0]

    return ttl_value

def get_os(ttl):

    ttl = int(ttl)

    if ttl >= 0 and ttl <= 64:
        return "Linux"
    elif ttl >= 65 and ttl <= 128:
        return "Windows"
    else:
        return "Not Found"

if __name__ == '__main__':

    ip_address = sys.argv[1]

    ttl = get_ttl(ip_address)

    os_name = get_os(ttl)
    print("\n%s (ttl -> %s): %s\n" % (ip_address, ttl, os_name))
```

## Mediante nmap

Como hemos mencionado, es posible llevarlo a cabo con el parámetro `-O` en [[nmap]].
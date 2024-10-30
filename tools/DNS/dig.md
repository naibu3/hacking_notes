Es una herramienta que nos permite llevar a cabo [[Ataques de transferencia de zona]].

# Sintaxis

```bash
dig @<DNS-server> <domain-name> AXFR
```

Donde:
- `DNS-server` es la dirección IP del servidor DNS que se desea consultar.
- `domain-name` es el nombre de dominio del cual se desea obtener la transferencia de zona.
- AXFR es el tipo de consulta que se desea realizar, que indica al servidor DNS que se desea una transferencia de zona completa. Aunque tenemos más opciones como:

	* `ns` para servidores [[DNS]].
	* `mx` para servidores de correo.
---
title: Subdomain enumeration
---

Uno de los pasos más importantes durante el reconocimiento es identificar todos los sitios web que posee la organización a auditar. Muchas organizaciones tienen un dominio principal ó *top-level domain*, mediante la búsqueda de subdominios podemos identificar otros recursos del sistema.

# Identificación pasiva

Es decir, aquella que se realiza sin interactuar directamente con el objetivo, sino tan sólo mediante fuentes abiertas. Una primera forma sería utilizando [[Google dorks]]:

```google
site: domain.com
```

Otra herramienta sería [dnsdumpster](https://dnsdumpster.com/), que utiliza datos de google, así como de otros sitios como [virustotal](https://virustotal.com).

Ó [crt.sh](https://crt.sh), que recolecta información del certificado.

Además hay otras herramientas para la terminal, como [[sublist3r]], (viene preinstalado con kali, pero puedes instalarlo desde su [github](https://github.com/aboul3la/Sublist3r)). Ésta toma información de DNS, pero debe utilizarse con cuidado ya que google suele bloquearla. O **amass**.
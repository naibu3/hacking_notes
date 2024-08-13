**Log Poisoning** es una técnica de ataque en la que un atacante **manipula** los **archivos de registro** (**logs**) de una aplicación web para lograr un resultado malintencionado. Esta técnica puede ser utilizada en conjunto con una vulnerabilidad **[[LFI]]** para lograr una **ejecución remota de comandos** en el servidor.

Como ejemplos para esta clase, trataremos de envenenar los recursos ‘**auth.log**‘ de **SSH** y ‘**access.log**‘ de **Apache**, comenzando mediante la explotación de una vulnerabilidad [[LFI]] primeramente para acceder a estos archivos de registro.

# Explotación

## Php

Si por ejemplo, en nuestra petición incluimos `<?php system("<comando>"); ?>` (ya sea como parámetro, en el ***user-agent***, ...), esto se guardará en */var/log/apache2/access.log*, de forma que si tenemos un **LFI**, al acceder, podremos ver el output.

## Ssh

En este caso, el archivo es */var/log/btmp* y el  input será el nombre de usuario.
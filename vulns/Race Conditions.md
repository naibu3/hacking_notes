#vuln 

# ¿Qué es?

Las **condiciones de carrera** (también conocidas como **Race Condition**) son un tipo de vulnerabilidad que puede ocurrir en sistemas informáticos donde dos o más procesos o hilos de ejecución compiten por los mismos recursos sin que haya un mecanismo adecuado de sincronización para controlar el acceso a los mismos.

Esto significa que si dos procesos intentan acceder a un mismo recurso compartido al mismo tiempo, puede ocurrir que la salida de uno o ambos procesos sea impredecible, o incluso que se produzca un comportamiento no deseado en el sistema.

Los atacantes pueden aprovecharse de las condiciones de carrera para llevar a cabo ataques de denegación de servicio (**DoS**), sobrescribir datos críticos, obtener acceso no autorizado a recursos, o incluso ejecutar código malicioso en el sistema.
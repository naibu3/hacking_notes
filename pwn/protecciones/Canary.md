#pwn #protections

# ¿Qué es?

Es una protección contra los ataques de [[Buffer overflow]]. Consiste en añadir un valor antes de la dirección de retorno.

![[canary.png]]

Esto funciona de forma que si se intenta sobrescribir el valor de la dirección de retorno, se detectaría un cambio en el valor del canary, produciéndose un error.

Todas las funciones de las librerías de C tienen esta protección. Cualquier *canary-found* hará detenerse al programa.
#vuln 

# ¿Qué es?

Las **Inyecciones CSS** (**CSSI**) son un tipo de vulnerabilidad web que permite a un atacante inyectar código CSS malicioso en una página web. Esto ocurre cuando una aplicación web confía en entradas no confiables del usuario y las utiliza directamente en su código CSS, sin realizar una validación adecuada.

El código CSS malicioso inyectado puede alterar el estilo y diseño de la página, permitiendo a los atacantes realizar acciones como la **suplantación de identidad** o el **robo de información confidencial**.

Las Inyecciones CSS (CSSI) pueden ser utilizadas por los atacantes como un vector de ataque para explotar vulnerabilidades de **Cross-Site Scripting** (**XSS**). Imagina que una aplicación web permite a los usuarios introducir texto en un campo de entrada que se muestra en una página web. Si el desarrollador de la aplicación no valida y filtra adecuadamente el texto introducido por el usuario, un atacante podría inyectar código malicioso en el campo de entrada, incluyendo código Javascript.

Si el código CSS inyectado es lo “suficientemente complejo”, puede hacer que el navegador web interprete el código como si fuera código JavaScript. Esto significa que el código CSS malicioso puede ser utilizado para inyectar código JavaScript en la página web, lo que se conoce como una inyección de JavaScript inducida por CSS (CSS-Induced JavaScript Injection).
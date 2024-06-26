#web 

# ¿Qué es?

Las **Insecure Direct Object References** (**IDOR**) son un tipo de vulnerabilidad de seguridad que se produce cuando una aplicación web utiliza **identificadores internos** (como números o nombres) para identificar y acceder a recursos (como archivos o datos) y no se valida adecuadamente la autorización del usuario para acceder a ellos.

Por ejemplo, supongamos que un usuario ‘**A**‘ tiene un pedido con el identificador numérico **123** y el usuario ‘**B**‘ tiene un pedido con el identificador numérico **124**. Si el atacante intenta acceder a través de la URL “**https://example.com/orders/124**“, la aplicación web podría permitir el acceso a ese pedido sin validar si el usuario tiene permiso para acceder a él. De esta manera, el atacante podría acceder al pedido del usuario ‘**B**‘ sin la debida autorización.
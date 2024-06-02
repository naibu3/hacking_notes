Cuando un cliente quiere conectarse a un servidor mediante **TCP**, tiene lugar lo que se conoce como el *three way handshake*.

El cliente comienza mandando un paquete ***SYN***, en caso de que el puerto esté abierto, el servidor responde con un ***SYN+ACK***, a lo que el cliente finalmente responde con un ***ACK***, conectándose.

En caso de que el puerto esté cerrado, el servidor responderá con ***RST+ACK***.
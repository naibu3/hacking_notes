El XML (Extensible Markup Language) es un formato simple basado en texto para representar la información de manera estructurada documentos, datos, configuraciones, libros, transacciones, facturas, y mucho más.

# Estructura

Un archivo XML tiene la siguiente estructura:

```xml
<?xml version="1.0" encoding="UTF-8"?>    -> Declaración XML
<!DOCTYPE foo [<!ENTITY xxe "miEntidad">]> -> DTD (Doc Type Definition)

<root>
	<name>hola</name>
	<tel>345345345</tel>
	<email>hola@hola.com</email>
	<password>hola</password>
</root>
```

# Entidades

Son una forma de representar un elemento de datos sin referenciar a los datos como tal. Funcionan de forma similar a una variable.

Por ejemplo, para la esytructura anterior, si cambiamos el valor de *email* por una referencia a la entidad *xxe*, ahora email pasará a valer **"miEntidad"**:

```xml
[...]
<!DOCTYPE foo [<!ENTITY xxe "miEntidad">]> -> DTD (Doc Type Definition)
[...]
	<email>&xxe</email>
[...]
```

Hay varios **tipos de entidades**:

- **Genéricas / customizadas**: Son totalmente personalizadas, se les pueden dar una clave y un valor, como si de una variable se tratara.

- **Externas (XXE)**: Cuando haces uso de un **DTD externo**. Las entidades externas ofrecen un mecanismo para dividir el documento XML en varias piezas en vez de usar un solo documento monolítico. Permite la reutilización de componentes en archivos separados y en recursos externos. Esto nos permite cargar archivos del sistema o incluso de servidores externos. Se usa la palabra "***SYSTEM***".

- **Predefinidas**: Son aquellas propias del propio XML, por ejemplo `&gt` y `&lt` para `>` y `<`.


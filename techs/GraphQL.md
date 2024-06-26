# ¿Qué es?

**GraphQL** es un lenguaje de consulta para **APIs** (**Application Programming Interfaces**) que se ha vuelto cada vez más popular en los últimos años. A diferencia de las APIs tradicionales que tienen endpoints fijos, GraphQL permite a los clientes solicitar sólo la información que necesitan y obtener una respuesta personalizada en función de sus necesidades.

# IDORs

En el contexto de GraphQL, los [[Insecure Direct Object Reference (IDOR)]] pueden ocurrir cuando un atacante es capaz de adivinar o enumerar identificadores (**IDs**) de objetos dentro de la API, y puede utilizar esos IDs para acceder a objetos a los que no debería tener acceso. Esto puede ocurrir porque los desarrolladores de la API pueden no haber implementado adecuadamente los mecanismos de autenticación y autorización en su API.

# Introspection

**Introspection** en GraphQL es un mecanismo que permite a los clientes **obtener información** sobre el esquema GraphQL de una API. Esto significa que los clientes pueden explorar y descubrir los tipos de datos, los campos y las relaciones que existen en la API, lo que puede ser muy útil para los desarrolladores que necesitan construir clientes GraphQL. Sin embargo, la introspección también puede ser utilizada por los atacantes para obtener información sensible sobre la estructura y los datos que existen en la API, lo que puede ser utilizado para llevar a cabo ataques más sofisticados.

# Mutations

Por otro lado, las **Mutations** en GraphQL son operaciones que permiten a los clientes **modificar los datos** en la API. A diferencia de las consultas, que sólo permiten la lectura de datos, las mutaciones permiten a los clientes agregar, actualizar o eliminar datos. Esto significa que las mutaciones tienen el potencial de ser utilizadas para realizar cambios importantes en la base de datos subyacente de la API. Si no se protegen adecuadamente, las mutaciones pueden ser explotadas por los atacantes para realizar cambios malintencionados en la API, como la eliminación de datos importantes o la creación de nuevos registros.
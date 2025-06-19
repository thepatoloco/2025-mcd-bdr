# Base de Datos Relacionales

## Tarea 1

### Instrucciones

- Crear un repositorio público de Github
- Compartir el repositorio en el Teams correspondiente (o Drive, mientras haya Teams)
- Describir una base de datos y sus relaciones de manera no estructurada (puede ser un párafo, lista, esquema...) con la que trabajar durante el tetramestre. Agrega el típo de dato que supones que tendrá cada uno de tus atributos.
- Investigar diferentes SGBD, elegir alguno y describirlo. Citar adecuadamente. Plagio invalida tarea.
- Subir esta descripción en un archivo markdown o PDF nombrado claramente (tarea 1 o algo por el estilo).

### Base de Datos estructurada

Se requiere crear una aplicación de mensajería grupal donde **usuarios** forman parte de **grupos** donde se envían **mensajes**. Los mensajes pueden tener imagenes y pueden ser respuestas a otros mensajes.

#### Usuario

- nombre (texto)

#### Grupo

- nombre (texto)
- Usuarios

#### Mensaje

- contenido (texto)
- Grupo
- Usuario
- imagen (binario)
- Mensaje respuesta
- timestamp (fecha y hora)

### Descripción SGBD

PostgreSQL es un SGBD que implementa varias características de la Programación Orientada a Objetos. Mientras que MySQL ofrece lo que normalmente se esperaría de un SGBD, como la creación de tablas y columnas con tipos de datos y relaciones con tipos de datos simples; PostgreSQL nos permite crear tipo de datos compuestos los cuales se pueden almacenar directamente en las columnas, nos permite la herencia de tablas, y nos permite almacenar arrays en las columnas (algo que en MySQL solamente se puede lograr mediante relaciones).

#### Bibliografía

- PostgreSQL y MySQL: diferencia entre los sistemas de administración de bases de datos relacionales (RDBMS). AWS. (s. f.). Amazon Web Services, Inc. <https://aws.amazon.com/es/compare/the-difference-between-mysql-vs-postgresql/>

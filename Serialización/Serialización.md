## TADs
Un TAD (tipo abstracto de dato) es una estructura de datos que tiene operaciones asociadas, que se encutnran bajo un mismo identificador. Generalmente en C, representamos un TAD mediante el uso de un ``struct``.

Cuando se trabaja con structs el tamaño de la memoria no siempre coincide con la suma del tamaño de sus elementos. Esto ocurre a cause del **padding** que agrega el compilador para intentar optimizar los accesos a memoria.

```C
// Estructura de datos
typedef struct {
    uint32_t dni;
    uint8_t edad;
    uint32_t pasaporte;
    char nombre[14];
} t_persona;

// Operaciones asociadas
t_persona persona_crear(uint32_t dni, uint8_t edad, uint32_t pasaporte, char *nombre) {
    t_persona persona;
    persona.dni = dni;
    persona.edad = edad;
    persona.pasaporte = pasaporte;
    strncpy(persona.nombre, nombre, 14);
    return persona;
}
```
![padding-imagen.png](\img\padding-imagen.png "Padding del tipo de dato")

El tamaño del padding depende de varios factores. Por esto, es que no tenemos garantias de cuánto espacio ocupará un struct en memoria.

Cuando querramos mandar mensajes a través de la red, nunca sabremos cuál es el tamaño real de nuestra estructura. Si vemos el prototipo de las funciones `send()` y `recv()`, podemos notar que ambas requieren que les digamos cuántos bytes vamos a enviar o recibir y si nosotros, usamos `sizeof(miTAD)` no tenemos garantías de que el tamaño de mi struct coincida entre el emisor y el destinatario, para evitar esto debemos *serializar*

## ¿Qué es serializacion?

El concepto consiste en especificar un protocolo en común, para que dos procesos se comuniquen de manera organizada. La idea es poder enviar un stream de datos siguiendo un orden definido y conocido por ambos procesos.

#### Estructuras estáticas

Supongamos que tenemos el siguiente TAD:
```C
typedef struct {
    uint32_t dni;
    uint8_t edad;
    uint32_t pasaporte;
    char nombre[14];
} t_persona;
```
Nuestro objetivo será organizar los elementos para enviarlos de una manera ordenada utilizando un protocolo:

![protocolo.jpg](\img\protocolo.jpg "Protocolo")

En este caso, un posible protocolo, es agregar un header que normalmente utilizaremos para indicar qué tipo de dato vamos a enviar. La idea es que del otro lado se reciba primero este header, y con eso ya se puede "preparar" para saber que viene (generalmente todo lo que viene despues del header se conoce como  *payload*)

###### ¿Como hacemos para serialziar este struct?
Primero empezamos reservando un bloque de HEAP, suficiente para almacenar nuestra estructura. éste lo denominaremos como nuestro buffer intermedio, donde se irán guardando nuestros datos mediante la funcion `memcpy()`.
De esta manera, se ignorará el padding que el compiladro crea, y se guardarán solamente los datos que necesitamos de nuestra estructura.
Una vez hecho esto, se podrá empaquetar el buffer y se enviará a través de la red.
Una vez que el destinatario los reciba, los desempaquetara de forma inversa (esto es posible dado que se determinó un protocolo).

#### Estructuras dinámicas

```C
typedef struct {
    char* username;
    char* message;
} t_package;
```
En esta estructura no tenemos un `char[14]`, tenemos un puntero `(char*)` que apunta a una posición de memoria provocando que haya estructuras que van a tener longitud variables, porque vamos a reservar memoria dinámicamente.
Si seguimos los pasos anteriores, el receptor de nuestra estructura no va a saber cuánta memoria deberá reservar para poder leer lo que recibió
Para resolver esto, se puede colocar el tamaño de bytes que necesitará el destinatario para almacenar lo que enviamos, es decir , teniendo la siguiente estructura, establecer el siguiente protocolo.

![protocolo-recomendado.jpg](\img\protocolo-recomendado.jpg "Protocolo recomendado")

```C
#include <commons/string.h>

typedef struct {
    char* username;
    uint32_t username_length;
    char* message;
    uint32_t message_length;
} t_package;

t_package package_create(char *username, char *message) {
    t_package package;
    package.username = string_duplicate(username);
    package.username_length = string_length(username) + 1; // +1 para el '\0'
    package.message = string_duplicate(message);
    package.message_length = string_length(message) + 1; // +1 para el '\0'
    return package;
}
```
![stack-heap.png](\img\stack-heap.png "Stack - Heap")
De esta forma, luego se proseguirá con los mismo passo que con una estructura estatica: hacemos `memcpy()` de los datos de nuestra estructura a un buffer intermedio, lo empaquetamos y lo enviamos. El recepetor cuando desempaqueta el paqueute, se fijará cuantos bytes debe reservar dinámicamente, y luego proseguirá a realizar el `memcpy()`

### Serialicemos
```C
typedef struct {
    uint32_t dni;
    uint8_t edad;
    uint32_t pasaporte;
    uint32_t nombre_length;
    char* nombre;
} t_persona;
```
Lo que haremos es usar un buffer temporal dondearmar nuestro paquete. En este caso tendremos:
- 4 bytes del dni
- 1 byte de la edad
- 4 bytes del pasaporte
- N bytes del nombre
- 4 bytes de la longitud (N) del nombre

El buffer podrias ser un simple `void*` reservado con `malloc()`. Sin embargo, para esta guia, vamos a armar un `struct t_buffer` que, además del puntero a nuestro stream de datos, tendra el tamaño en bytes del mismo stream. Esto nos permitirá poder recibir el payload entero desde el lado receptor en una sola operación, puesto que conoceremos su tamaño.

```C
typedef struct {
    uint32_t size; // Tamaño del payload
    uint32_t offset; // Desplazamiento dentro del payload
    void* stream; // Payload
} t_buffer;

```

Para ir llenando este buffer, como vimos anteriormente, utilizaremos la funcion `memcpy()`, que nos permite copiar bytes desde un bloque de memori origen a uno de destino:
```C
void *memcpy(void *dest, const void *src, size_t n);
```
Ahora, si tenemos una variable declarada `(t_persona persona)` con los datos a enviar en ella, entonces llenar nuestro buffer podría ser algo como los siguiente:

```C
typedef struct {
    uint32_t size; // Tamaño del payload
    uint32_t offset; // Desplazamiento dentro del payload
    void* stream; // Payload
} t_buffer;
```
Para ir llenando este buffer utilizaremos la funcion `memcpy()`, que nos permite copiar bytes desde un bloque de memoria origen a uno de destino:
```C
void *memcpy(void *dest, const void *src, size_t n);
```

Ahora, si tenemos una variable declarada `(t_persona persona)` con los datos a enviar en ella, entonces llenar nuestro buffer podría ser: 
```C
t_buffer* buffer = malloc(sizeof(t_buffer));

buffer->size = sizeof(uint32_t) * 3 // DNI, Pasaporte y longitud del nombre
             + sizeof(uint8_t) // Edad
             + persona.nombre_length; // La longitud del string nombre.
                                      // Le habíamos sumado 1 para enviar tambien el caracter centinela '\0'.
                                      // Esto se podría obviar, pero entonces deberíamos agregar el centinela en el receptor.

buffer->offset = 0;
buffer->stream = malloc(buffer->size);

memcpy(stream + offset, &persona.dni, sizeof(uint32_t));
buffer->offset += sizeof(uint32_t);
memcpy(stream + offset, &persona.edad, sizeof(uint8_t));
buffer->offset += sizeof(uint8_t);
memcpy(stream + offset, &persona.pasaporte, sizeof(uint32_t));
buffer->offset += sizeof(uint32_t);

// Para el nombre primero mandamos el tamaño y luego el texto en sí:
memcpy(stream + offset, &persona.nombre_length, sizeof(uint32_t));
buffer->offset += sizeof(uint32_t);
memcpy(stream + offset, persona.nombre, persona.nombre_length);
// No tiene sentido seguir calculando el desplazamiento, ya ocupamos el buffer completo

buffer->stream = stream;

// Si usamos memoria dinámica para el nombre, y no la precisamos más, ya podemos liberarla:
free(persona.nombre);
```

Para ir moviendonos en el buffer utilizamos una variable de desplazamiento (*offset*). La idea es que si tenemos un puntero, podemos "sumarle/restarle" valores y eso nos da otro puntero. De esta manera, haciendo `buffer + offset` nos desplazamos tantos bytes respecto del inicio del buffer como indique nuestra variables `offset`.

Una vez cargado el buffer del TAD persona, podemos agregarle un header al principio para "empaquetarlo". De esta manera el receptor viendo el header ya sabe que le están mandando una persona y se puede prepara para recibir el resto de datos.

```C
typedef struct {
    uint8_t codigo_operacion;
    t_buffer* buffer;
} t_paquete;
```

Entonces podemos llenar el paquete con el buffer

```C
t_paquete* paquete = malloc(sizeof(t_paquete));

paquete->codigo_operacion = PERSONA; // Podemos usar una constante por operación
paquete->buffer = buffer; // Nuestro buffer de antes.

// Armamos el stream a enviar
void* a_enviar = malloc(buffer->size + sizeof(uint8_t) + sizeof(uint32_t));
int offset = 0;

memcpy(a_enviar + offset, &(paquete->codigo_operacion), sizeof(uint8_t));

offset += sizeof(uint8_t);
memcpy(a_enviar + offset, &(paquete->buffer->size), sizeof(uint32_t));
offset += sizeof(uint32_t);
memcpy(a_enviar + offset, paquete->buffer->stream, paquete->buffer->size);

// Por último enviamos
send(unSocket, a_enviar, buffer->size + sizeof(uint8_t) + sizeof(uint32_t), 0);

// No nos olvidamos de liberar la memoria que ya no usaremos
free(a_enviar);
free(paquete->buffer->stream);
free(paquete->buffer);
free(paquete);
```

Una vez enviado el mensaje ¿Como deserializamos desde el receptor?

```C
t_paquete* paquete = malloc(sizeof(t_paquete));
paquete->buffer = malloc(sizeof(t_buffer));

// Primero recibimos el codigo de operacion
recv(unSocket, &(paquete->codigo_operacion), sizeof(uint8_t), 0);

// Después ya podemos recibir el buffer. Primero su tamaño seguido del contenido
recv(unSocket, &(paquete->buffer->size), sizeof(uint32_t), 0);
paquete->buffer->stream = malloc(paquete->buffer->size);
recv(unSocket, paquete->buffer->stream, paquete->buffer->size, 0);

// Ahora en función del código recibido procedemos a deserializar el resto
switch(paquete->codigo_operacion) {
    case PERSONA:
        t_persona* persona = persona_serializar(paquete->buffer);
        ...
        // Hacemos lo que necesitemos con esta info
        // Y eventualmente liberamos memoria
        free(persona);
        ...
        break;
    ... // Evaluamos los demás casos según corresponda
}

// Liberamos memoria
free(paquete->buffer->stream);
free(paquete->buffer);
free(paquete);
```

La funcion para deserilzar el payload quedaria:

```C
t_persona* persona_serializar(t_buffer* buffer) {
    t_persona* persona = malloc(sizeof(t_persona));

    void* stream = buffer->stream;
    // Deserializamos los campos que tenemos en el buffer
    memcpy(&(persona->dni), stream, sizeof(uint32_t));
    stream += sizeof(uint32_t);
    memcpy(&(persona->edad), stream, sizeof(uint8_t));
    stream += sizeof(uint8_t);
    memcpy(&(persona->pasaporte), stream, sizeof(uint32_t));
    stream += sizeof(uint32_t);

    // Por último, para obtener el nombre, primero recibimos el tamaño y luego el texto en sí:
    memcpy(&(persona->nombre_length), stream, sizeof(uint32_t));
    stream += sizeof(uint32_t);
    persona->nombre = malloc(persona->nombre_length);
    memcpy(persona->nombre, stream, persona->nombre_length);

    return persona;
}
```
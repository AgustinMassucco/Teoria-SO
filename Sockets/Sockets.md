# Sockets
## ¿Que es un cliente - servidor?

El propósito del cliente es iniciar una conexión contra el servidor para realizar solicitudes, y este ultimo es el que este va a entender y resolver. Si el servidor no está andando, el cliente no puede realizar ningún tipo de solicitud.
Por otro lado, es responsabilidad del servidor aceptar las conexiones entrantes de los muchos clientes, es decir, ser capaz de atender varios clientes a la vez, y mantener disponibilidad de sus servicios una vez se haya terminado la solicitud de un cliente

## Ips y puertos

Cuando tenemos dos o más computadoras dentro de una misma red, cada dispositivo tiene una IP, un ID único para cada dispositivo dentro de la red. Las IPs tienen la particularidad de ser cuatro números, separados por un punto, donde cada uno va de 0 a 255. Esa es la IP que la computadora tiene asignada dentro de nuestra red doméstica.
Luego tenemos los puertos. Estos son unidades lógicas dentro del sistema operativo para que una misma computadora, y por lo tanto la misma IP, pueda tener varios procesos corriendo que necesiten operar dentro de la red.
Cada sistema operativo tiene 65535 puertos (2^16), y tiene los primeros 1000 reservado para sus tareas y otras cosas. Lo importante es que cuando desarrollemos la comunicación entre nuestros procesos, debemos tomar puertos que estén libres porque dos procesos no pueden ocupar el mismo puerto al mismo tiempo.

## Protocolo de comuncación

Un protocolo de comunicación no es más que una definición y estandarización de como se van a estar comunicando dos procesos distintos en la red.  Para que dos procesos se puedan comunicar, necesitan estar utilizando el mismo protocolo de comunicación.
Cuando dos procesos van a conectarse, lo primero es realizar una operación llamada “handshake”. Es una presentación por parte del cliente para hacerle saber al servidor que cumplen el mismo protocolo, y por lo tanto pueden entablar una comunicación. 
Por otro lado, también es responsabilidad del servidor detectar cuando se le quiere conectar un cliente de otro protocolo para rechazar esa conexión.

## Sockets

Un socket es la representación que el sistema operativo le da a una conexión. Cuando un cliente quiere iniciar una conexión contra su servidor, lo que hace es solicitarle al sistema operativo que cree un socket, hacer que ese socket ocupe un puerto libre, y se conecte con el socket del servidor que está a la espera de nuevos clientes, ocupando también un puerto del servidor.
Las llamadas al sistema que el cliente tiene que realizar para conectarse a su servidor son distintas a las que el servidor tiene que hacer para prepararse para recibir conexiones entrantes de los clientes.

![cliente-servidor.png](\Imagenes\cliente-servidor.png "Flujo de comunicacion")

## socket()
La primera syscall que hay que utilizar para iniciar una conexión por socket entre dos procesos es ***socket()***. Esta lo que hace es generar lo que se llama un *file descriptor* (fd), que son básicamente los IDs que Linux utiliza para representar cualquier cosa del sistema. Estos fd son representados en los programas C por un entero. Para poder crear un socket cliente y un servidor corriendo en la misma máquina, podemos hacerlo de la siguiente manera:
##### Cliente
``` C
int err;

struct addrinfo hints, *server_info;

memset(&hints, 0, sizeof(hints));
hints.ai_family = AF_INET;
hints.ai_socktype = SOCK_STREAM;

err = getaddrinfo("127.0.0.1", "4444", &hints, &server_info);

int fd_conexion = socket(server_info->ai_family,
                         server_info->ai_socktype,
                         server_info->ai_protocol);

// ...

freeaddrinfo(server_info);
```

##### Servidor
``` C
int err;

struct addrinfo hints, *server_info;

memset(&hints, 0, sizeof(hints));
hints.ai_family = AF_INET;
hints.ai_socktype = SOCK_STREAM;
hints.ai_flags = AI_PASSIVE;

err = getaddrinfo(NULL, "4444", &hints, &server_info);

int fd_escucha = socket(server_info->ai_family,
                        server_info->ai_socktype,
                        server_info->ai_protocol);

// ...

freeaddrinfo(server_info);
```
`getaddrinfo()` es una llamada al sistema que devuelve informacion de red sobre la IP y puerto que le pasemos, en este caso del servidor. Nos guarda en la variable `server_info` un **puntero** que apunta hacia lso datos necesarios apra la creacion del socket. Esta memoria que nos fue devuelta debe ser liberada con `freeaddrinfo()`.

 - Si el flag `AI_PASSIVE` está presente en `hints.ai_flags` y el primer parámetro de `getaddrinfo()` es `NULL`, las direcciones devueltas por `getaddrinfo()` son las adecuadas para crear un socket de **escucha**.
 - Por el contrario, si no está presente el flag `AI_PASSIVE` y el primer parámetro de `getaddrinfo()` es una dirección IP, las direcciones devueltas por `getaddrinfo()` son las adecuadas para crear un socket de conexion.
  
Para el caso particular de `socket()` quedemos en que esta configuración nos crea un socket capaz de hacer todo lo que vamos a necesitar.

#### TIP

Se puede verificar la documentacion de cada funcion para revisar en detalle los valores de retorno y manejo de sus errore. Por lo general, a menos que se pida alguna lógica de reintentos, lo más comín va a ser imprimir por pantalla el error y finalizar el programa con `abort()`.

## bind() y listen()

Los clientes, para poder comunicarse con sus servidores, deben hacerlo a través de la IP del servidor físico en el que estén corriendo y el puerto fisico que estén ocupando en dicho servidor a la espera de nuevas conexion es por parte de los clientes.
Las llamadas al sistema que realizan esas preparaciones por parte del proceso servidor son `bind()` y `listen()`.

  - Primero, `bind()` toma el socket que creamos con anterioridad y le pide al sistema operativo que lo asocie al puerto que le digamos

Esa petición probablemente falle si uno inicia un servidor después de haber finalizado otro en el mismo puerto.
Esto se debe a que el sistema operativo no libera el puerto inmediatamente por razones de seguridad. La forma de aliviarlo es agregar una configuración mediante la función `setsockopt() : SO_REUSEPORT`. Esta opción permite que varios sockets se puedan *bindear* a un puerto al mismo tiempo, siempre y cuando pertenezcn al mismo usuario.

- Luego, `listen()` toma ese mismo socket y lo marca en el sistema como un socket cuya **única responsabilidad** es notificar cuando un nuevo cliente esté intentando conectarse.

##### Servidor
```C
err = setsockopt(fd_escucha, SOL_SOCKET, SO_REUSEPORT, &(int){1}, sizeof(int));

err = bind(fd_escucha, server_info->ai_addr, server_info->ai_addrlen);

err = listen(fd_escucha, SOMAXCONN);
```

`bind()` está recibiendo le puerto que debe ocupar a partir de los datos que le suministramos al `getaddrinfo()` con anterioridad.
Luego, `listen()` recibe como segundo parámetro la cantidad de conexiones vivas que puede mantener. `SOMAXCONN` es la cantidad máxima que admite el sistema operativo.

## accept() y connect()

Una vez el socket del servidor se marcó en modo de **escucha**, estamos preparados para empezar a recibir las conexiones de nuestros clientes

Para esto, el servidor utiliza la llamada al sistema `accept()`, la cual es bloqueante. Esto significa que el proceso servidor se quedará bloqueado en `accept()` hasta que se le conecte un cliente.
Cuando el cliente intente conectarse al servidor, lo hará mediante la llamada al sistema `connect()`. Si el servidor no está en `accept()`, `connect()` fallará y devolveá un error.

##### Cliente
```C
err = connect(fd_conexion, server_info->ai_addr, server_info->ai_addrlen);
```

##### Servidor
```C
int fd_conexion = accept(fd_escucha, NULL, NULL);
```

Una vez que el cliente fue acceptado, `accept()` retorna un **nuevo** socket (file descriptor) que representa la conexión **bidireccional** entre ambos procesos.

Esto quiere decir nuestro `fd_escucha` no va a ser el que participe de dicha comunicación, solamente tiene la responsabilidad de quedarse escuchando nuevas conexiones y aceptarlas.

## send() y recv()

Una vez establecida la conexion esntre el cliennte y el servidor, ya estamos listos para comenzar a enviar mensajes libremente entre ambos. Pero antes, un último paso que se recomienda cumplir es el proceso que se conoce como **handshake**.
El único proposito de este proceso es enviar un paquete que le informe al servidor cual es el protocolo con el que está intentando iniciar una conversación para el servidor le conteste si es capaz de enter ese protocolo o l einforme que no, en caso contrario

###### Ejemplo
Tomemos un servidor super minimalista donde su handshake es que el cliente le envie un `int32_t` (entero signado de 32 bits) cuyo valor sea 1. Digamos tambíen por simpleza que cuando el servidor responde 0 es OK y -1 es ERROR

###### Cliente
```C
size_t bytes;

int32_t handshake = 1;
int32_t result;

bytes = send(fd_conexion, &handshake, sizeof(int32_t), 0);
bytes = recv(fd_conexion, &result, sizeof(int32_t), MSG_WAITALL);

if (result == 0) {
    // Handshake OK
} else {
    // Handshake ERROR
}
```

###### Servidor
```C
size_t bytes;

int32_t handshake;
int32_t resultOk = 0;
int32_t resultError = -1;

bytes = recv(fd_conexion, &handshake, sizeof(int32_t), MSG_WAITALL);
if (handshake == 1) {
    bytes = send(fd_conexion, &resultOk, sizeof(int32_t), 0);
} else {
    bytes = send(fd_conexion, &resultError, sizeof(int32_t), 0);
}
```

`recv` en este caso es bloqueante debido a que le estamos pasando el flag `MSG_WAITALL`, que se encarga de esperar a que llegue po socket la cantidad de bytes que le decimos en el tercer parametro. A causa de esto, el cliente espera la respuesta del handshake antes de continuar. 
Por otro lado, si el cliente en lugar de enviar un 1, estuviera enviando otro entero, o un `char` con el valor "1", el servidor le devolveria error, porque entiende que está solicitando comunicarse mediante otro protocolo.
Durante este proceso, en `bytes` nos vamos guardando la cantidad de bytes enviados y/o recibidos en ambos lados. Esto nos va a servir para ir validanod que efectivamebte se hayan enviado y recibido los bytes esperados y manejar los casos de error o de desconexión.
Una vez pasado el proceso de handshake, ya el cliente se encuentra en vía libre para poder enviarle otros mensajes al servidor para que éste le conteste con los resultados de sus solicitudes. Tanto `send()` como `recv()` se encargan de mover bytes de datos a través de la red, y eso es lo que representa el segundo parámetro de ambas funciones.

## close()
Por último los socets una vez que no los usemos más deben ser cerrados con `close(fd)`. Esto normalmente se realiza con los `fd_conexion` ya que los `fd_escucha` deben dar disponibilidad constante para todas las solicitudes de los clientes que tenga en el tiempo en el que el proceso esté ejecutando.

Cuando uno de los dos nodos se desconecta, el otro podria estar esperando un mensaje o intentar enviar uno. De ser así ambas syscalls se manejan de forma diferente.
 - `recv` retorna el valor 0 para que podamos manejar la desconexion y cerrar el socket.
 - `send` lanza una señal llamada `SIGPIPE` que interrumpe la ejecución del programa. Si queremos evitar esto, podemos pasarle el flag `MSG_NOSIGNAL` como último parámetro al momento de invocarla

## Multiplexando
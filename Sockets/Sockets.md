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

![flujo comunicacion](\Imagenes\cliente-servidor.png "Flujo de comunicacion")

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
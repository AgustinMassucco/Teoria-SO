## Repaso de Arquitectura
#### Componentes básicos de una computadora
 !["Arquitectura de un computador"](Img\Arquitectura-de-un-computador.png)

 **Procesador**: Está formado por un conjunto de registros que almacenan datos, una unidad aritmética (ALU) que realiza operaciones con los datos y una unidad de control que coordina a todos los componentes.

 **Registros**: Es una porcion de memoria donde el SO almacena información. Se dividen en 2 categorias:
 - *Registros visibles por el ususario (de uso general)*: Estos registros pueden ser modificados directa o indirectamente (leer o escribir)(AX,BX,etc.).
 - *Registros de control y estado*: generalmente no pueden ser modificados por un programador pero sí consultados. Los utiliza el hardware o software para funcionar y los modifica de acuerdo a la ultima instruccion que ejecutó
   - PC: Contiene la dirección de la próxima instruccion a ejecutarse.
   - IR: guarda la instruccion en sí que se está ejecutando en ese momento, pero no la direccion
   - MAR: contiene la direccion del proximo operando a ser utilizado por la instruccion.
   - MBR: Guarda la direccion del próximo operando a ser utilizado por la instruccion.
   - PSW: Indica el estado luego de realizar una operación (overflow, carry, flags).

**Memoria RAM**: Conjunto de direcciones lineaes. Es un gran array de información

**BUS**: Dispositivo capaz de transferir datos entre los distintos componentes de una computadora. Podemos decir que permite la comunicacion entre el procesador, dispositivos I/O y memoria principal. Está formado por:
- Bus de datos
- Bus de direccioes
- Bus de control

#### Instrucciones

Cada procesador tiene un set de instrucciones qe son simplemente las funciones que realiza el procesador, como MOV, ADD, SUB, HLT, etc.

*i = i + 1* No es una instruccion. A esto se lo llama *sentencia*. Esta sentencia lleva por detras tres instrucciones que deberá realizar el procesador.

 !["Proceso de una sentencia"](Img\Proceso-de-una-sentencia.png)

**Tipos de instrucciones**: Existe una extensa clasificacion de las instrucciones pero lo único que nos importa saber es que hay instrucciones que sólo puede ejecutarlas el SO
- *Instrucciones no privilegiadas*: Cualquier programa puede ejecutarlas. Ej: MOV, ADD, SUB, JNZ, JZ, CALL.
- *Instrucciones privilegiadas*: Solo el SO puede ejecutarlas. Ej: CLI, STI, INT, HIT.

**Ciclo de instrucción sin interrupciones** Este es el ciclo básico de instruccion que no contempla la aparición de interrupciones durante la ejecucion.

!["Ciclo de una instruccion"](Img\Ciclo-de-una-instruccion.png)

Hay que tener en cuenta que sólo los programas que están en la memoria RAM pueden ejecutarse y que, en realidad, este ciclo está compuesto por 6 etapas pero en el gráfico y en la cursada solo contemplamos 3:
- **Fetch**: El procesador va a buscar a memoria la siguiente instrucción que tiene que ejecutar. Para saber cual es debe consutar el PC(Program counter).
- **Decode**: Decodifica la instruccion, es decir, la traduce para encontrar que operacion es y en una etapa siguiente, busca los operandos. La guarda en el registro IR.
- **Execute**: El procesador ejecuta la instruccion.

Una vez que se completa este ciclo para una instrucción el ciclo vuelve a empezar pero ahora con la siguiente instruccion

#### Interrupciones

Una interrupcion es un mecanismo del hardware mediante el cual se da aviso a la CPU de que ha ocurrido un evento. Habitualmente, la interrupcion proviene de algun módulo de I/O, de la memori o de la misma CPU. Cuando un programa es interrumpido se guardan todos los registros dek procesador (a esto se lo llama "contexto de ejecucion de un proceso", es decir, se guarda el contexto de ejecucion del proceso) antes de pasar a ejecutar la rutina de interrupción. Entonces, se interrumpe la ejecución normal de un programa para ejecutar las instrucciones que indique la rutina de interrupcion (esto siempre implica un *cambio a modo kernel* ya que este es el encargado de manejar las interrupciones). Una vez que finaliza la rutina el SO va a decidir que otro programa va a ejecutar. Todas las interrupciones deben ser atendidas en algún momento.

!["Ciclo de una interrupcion"](Img\Ciclo-de-una-interrupcion.png)
###### Clasificacion
- *Hardware/Software*
  - Hardware: Provienen de cualquier lado menos de la CPU. Son externas al procesador.
  - Software: Son generadas por la propia CPU, como consecuencia del ciclo de ejecución. Son internas.
- *Enmascarables/No enmascarables*:
  - Enmascarables: Pueden ser ignoradas temporalmente. Estas interrupciones no son atendidas inmediatamente cuando ocurren. No representan algo crítico dentro del sistema.
  - No enmascarables: Tienen que ser atendidas inmediatamente. Representan algo critico dentro del sistema. Están asociadas a elementos de hardware como la memoria o el disco y quien dispara esta interrupción es el dispositivo que está teniendo problemas.
- *Sincrónicas/Asincrónicas*:
  - Sincronicas: Son generadas por la CPU al ejecutar instrucciones, son internas.
  - Asincronicas: Son generadas por dispositivos externos al procesador. Pueden producirse en cualquier momento, independientemente de lo que esté haciendo el procesador. Son externas
- *E/S*: Son interrupciones que indican que finalizó un evento asociado a un dispositivo de E/S, como una escritura en disco, lectura de la placa de red, etc.
- *Fallas de Hardware*: Son no enmascarables
- *De clock*: Son interrupciones que le indican a la CPU que debe desalojar al proceso que está ejecutando actualmente. Son usadas par amanejar la planificacion de la CPU y permitir la ejecución de varios programas de manera concurrente
- *Excepciones*: Son generadas por errores en la programacio o por condiciones anomalas. Son las de más alta prioridad.
  - *Aborts*: un error grave ocurrió como fallo de HW
  - *Fallos*: Pueden ser corregidos y retoman la ejecución
  - *Traps*: son utilizadas para debugging.

##### Ciclo de instruccion con interrupciones
!["Ciclo de una instruccion con interrupciones"](Img\Ciclo-de-una-instruccion-interrupciones.png)

En **interrupt stage** se pregunta si hubo una interrupcion no enmascarable. Si la hubo, se interrumpe inmediatamente el ciclo de instrucciony el PC pasa a apuntar a la rutina de atención de la interrupcion. Si no hubo interrupciones no enmascarables, se pregunta si als enmascarables están habilitadas. Si no lo están se continúa con el ciclo normal de ejecucion.
Es importante remarcar que la interrupcion pudo habberse mandado en cualquier instante (fetch, decode, execute), pero estas etapas son inseparables. Recién se va a pregutnar si hubo una interrupcion despues de ejecutar la instrucción.

**Pasos que e generan cuando ocurre la interrupcio**
- Hardware (procesador):
  1) Se genera la interrupcion en cualquier momento.
  2) Finaliza la instruccion actual normalmente.
  3) Se determina que, efectivamente, ha ocurrido una interrupcion en la ejecucion y se identifica de que dispositivo provino
  4) se guarda el PC y PSW del programa que se estaba ejecutando.
  5) Se c arga en el PC la direccion del *Manejador de interrupciones* y comienza a ejecutar el SO.
- SO (Toma el control el manejador de interrupciones)
  1) Guarda todo el resto de la informacion que estaba en el procesador (registros AX, BX, registros de estado, etc.)
  2) Se inhabilitan las interrupciones
  3) Se procesa la interrupcion.
  4) Una vez que se hayan ejecutado todas las instrucciones del manejador de interrupciones, se restaura la informacion guardada del procesador
  5) Se restaura el PC y el PSW (como resultado, la próxima instruccion que se ejejcute va a ser la que le seguía al progra interrumpido). Siempre se restaura primero el PSW porque apenas se restaure el PC la CPU comenzara a ejecutar.
  6) Se habilitan nuevamente las interrupciones.

**Multiples interrupciones**: Ocurre cuando surge una interrupción mientras se está atendiendo otra. Se pueden seguir tres estrategias.
- Se atienden en orden secuencial.
- Se las ordena por prioridades. Para implementar esta alternativa se deberá definir una prioridad para cada interrupción. Se permitirá que una interrupcion de mayor prioridad interrumpa el procesamiento de una interrupción de menor prioridad.
- Deshabilitar las interrupciones mientras una interrupcion está siendo procesada.

#### Jerarquía de Memoria
Una computadora tiene mucho espacio de distinto tipo o de distintas características para almacenar informacion. EN el gráfico podemos ver varios. Los de la base son más lentos, tienen más capacidad y son menos costosos. Los de la cima son más rapidos, más pequeños pero más costosos.    

!["Ciclo de una instruccion con interrupciones"](Img\Jerarquia-memorias.png)

*Volatil* = Al apagar la computadora toda la infromacion almacenada se borra.
*No volatil* = Al apagar la computadora los archivos se mantienen.
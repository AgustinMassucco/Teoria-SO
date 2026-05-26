<span style="font-family: Luciole">

# Memoria Virtual

---

## Memoria Principal

- A nivel HW se refiere particularmente a la RAM.
- A nivel estructura podemos pensarlo como una <u>*gran matriz*</u> que simplemente posee <u>*direcciones*</u>
- Todas las **instrucciones** que se ejecutan y los **datos** sobre los que se operan tienen que estar en la memoria principal (física).

Esto se debe a que la memoria principal y los registros de cada CPU son los únicos medios de almacenamiento a donde la CPU puede acceder de forma directa. Existen instrucciones de máquina que toman direcciones de memoria como argumentos, pero no existe ninguno que toma direcciones de disco como argumentos. Si los datos sobre los cuales se operan no se encuentran en memoria principal, se tendrá que moverlos a algunos de estos medios de almacenamiento antes de que la CPU opere con ellos.

![Memoria Principal](./img/Memoria_repartida.png)

- Tendrá un espacio reservado para el SO (*Kernel space*).
- Tendrá un espacio reservado para repartirlo entre distintos usuarios (*User space)*.

**Observación**: Una aclaración es que se hará la simplificación de toda la memoria en el *user space* (para simplificar cuentas). Siempre habrá una porción de memoria que la usará el SO. Estará separada pues el SO puede que le convenga utilizar otra estrategia de cómo guardar/administrar la información respecto a lo que se utilicen en los procesos.

![Memoria de los registros](./img/Register_memory.png)

- Base register (Relocation register): Almacena la dirección de memoria física legal más pequeña.
- Limit register: especifica el tamaño del rango.

*Ej.*: Si 300040 es la base y 120990 es el límite $\rarr$ el programa puede acceder en forma legal direcciones desde 300040 hasta 420939 (inclusive)

Estos registros solamente los puede sobrescribir el SO, el cual utiliza una instrucción privilegiada que solamente pueden ser ejecutadas en modo kernel y el SO es el único quien ejecuta en modo kernel. Por lo tanto es  el único quien puede cargar los registros base y límite, a su vez que previene que el programa usuario cambie los contenidos de estos registros.

El SO, ejecutando en modo kernel, recibe acceso irrestricto sobre la memoria del SO y de los usuarios. Esto permite que el SO cargue programas de usuarios en el espacio de memoria de los usuarios, para terminar dichos programas en casos de error, para acceder y modificar parámetros de syscalls, para llevar a cabo I/Os desde y hacia memoria de usuario, entre otros.

### Protección del espacio de memoria

Esto se logra haciendo que la CPU compare cada dirección generado en modo usuario con los registros. Cualquier intento de un programa (ejecutando en modo usuario) queriendo acceder al espacio de memoria del SO o a los espacio de memoria de otros programas usuarios evocará en un trap hacia el SO (el cual se lo trata como *fatal error*).

![Memory protection](./img/Memory_trap.png)

## Velocidad de acceso a los medios de almacenamiento (respecto de la CPU)

![Velocidad de acceso a memoria](./img/Velocidad_de_acceso_a_memoria.png)

- Registros: almacenamiento más inmediato.
- Caché: Almacenamiento intermedio que posee un subconjunto de datos (una información reducida empleada frecuentemente para poder agilizar los accesos).
- RAM: se cargan los procesos en memoria para poder ejecutarlos
- Disco (referido al rígido): posee el problema de performance que es mecánico (rotaciones del disco)

## Requerimientos

Requerimientos que debe cumplir el SO para dar algún servicio respecto a memoria (tener el programa en memoria para poder ejecutarlo).

Funciones que debe satisfacer:

- Realocación: ¿Qué tan fácil será que durante el tiempo de ejecución el SO, para optimizar, pueda ir moviendo un proceso de un lugar a otro?¿Se podrá asignarle un bloque de memoria distinto al que se asignó antes de suspenderlo? 
  
Ej.: El planificador de mediano plazo se encarga del swapping (pasar procesos a un espacio de almacenamiento secundario -disco- para poder liberar espacio y viceversa)

- Protección: ¿Qué sucede si un proceso intenta acceder a una dirección de memoria que no es del proceso?
  
El So debería ser capaz de brindar protección (delimitar los segmentos de memoria del proceso). Definir un espacio de direcciones para que solo sea accesible por el proceso "padre"

- Compartir memoria: a veces se requiere que un proceso ejecute en forma colaborativa con otros procesos. Los procesos deben expresar la compartición de memoria en forma explícita ante el SO.

**Respecto a cómo se interpretará la memoria**:

- Organización lógica: cómo se la ve lógicamente a nivel estructura.
- Organización física: cómo se guardará efectivamente en la memoria.

![Requerimientos](./img/Requerimientos.png)

## Reasignación de direcciones (Adress Binding)

- Programa: un programa reside en disco como un archivo binario ejecutable. Para que pueda ser ejecutado debe ser cargado en memoria y ser colocado dentro del contexto de un proceso.

Las direcciones de memoria en el código fuente son generalmente **simbólicas** (puntero/posición enésimo de un vector). No representan una dirección en particular en la memoria ¿En qué momento se reasigna dicha dirección *simbólica*? Clásicamente, la reasignación de instrucciones y datos a direcciones de memoria puede darse en alguno de los siguientes pasos:

pag 8

</span>
<span style="font-family: Luciole">

# Memoria Real

## Introducción

- Un programa debe cargarse en memoria desde disco y colocarse dentro de un proceso para que se ejecute.
- La memoria principal y los registros son los únicos dispositivos de almacenamiento a los que puede acceder la CPU directamente.
- El acceso a registro es muy rápido; supone un siclo de CPU (o menos)
- El acceso a memoria principal puede durar varios ciclos
- Las memorias **caché** se colocan entre la memoria principal y la CPU para acelerar el acceso a la información

## Organización física de la memoria.

![Organización física de la memoria](./img/Organización_Física_de_la_memoria.png)

## Organización lógica de la memoria

- La memoria principal es un arreglo de palabra o bytes, cada uno de los cuales tiene una dirección (espacio de direcciones).
- La interacción es lograda a través de un conjunto de lecturas y escrituras a direcciones especificas realizadas por los procesos.

![Organización Lógica de la memoria](./img/Organización_Lógica_de_memoria.png)

## Procesos y Memoria

- Para que un proceso se ejecute se requiere ubicarlo en memoria principal junto con los datos que direcciona.
- Para optimizar el uso del computador se requiere tener varios procesos en memoria principal. (grado de multiprogramación)

![Procesos y Memoria](./img/Procesos_y_memoria.png)

# Memoria Virtual

La memoria principal es pequeña como para acomodar todos los programas y datos permanentemente. Por lo que es necesario implementar mecanismos de *memoria virtual*. La *memoria virtual* es una técnica para dar la ilusión de tener más memoria que la memoria principal.

## Vinculación de direcciones

La vinculación de instrucciones y datos a direcciones de memoria puede realizarse en tres etapas diferentes:

- **compilación**: Si se conoce a priori la posición que va a ocupar un proceso en la memoria se puede generar **código absoluto** con referencias absolutas a memoria; si cambia la posición del proceso hay que recompilar el código.
- **Carga**: Si no se conoce la posición del proceso en memoria en tiempo de compilación se debe generar **código reubicable**
- **Ejecución**: Si el proceso puede cambiar de posición durante su ejecución la vinculación se retrasa hasta el momento de ejecución. Necesita soporte hardware para el mapeo de direcciones (ej: registro base y limite).

## Espacio de Direcciones

El concepto de espacio de *direcciones lógicas* vinculado a un *espacio de direcciones físicas* separado es crucial para una buena gestión de memoria.

- **Dirección lógica** - es la dirección que genera el proceso; también se conoce como *dirección virtual*.
- **Dirección física** - dirección que percibe la unidad de memoria.

Las direcciones lógicas y físicas son iguales en los esquemas de vinculación en tiempo de compilación y de carga; pero difieren en el esquema de vinculación en tiempo de ejecución

## Base y límite

Un par de registros **base** y **límite** definen el espacio de direcciones lógicas

![Base y límite](./img/base_y_limite.png)

# Memory Managment Unit (MMU)

La *MMU* es un dispositivo hardware que transforma las direcciones virtuales en físicas. Con la MMU el valor del registro *reubicado* (registro base) es añadido a a cada dirección generada por un proceso de usuario en el momento en que es enviada a la memoria.
El programa de usuario trabaja con direcciones *lógicas*, nunca ve las direcciones *físicas* reales.

![MMU](./img/MMU.png)

## Administrador de memoria

| *Sistema monoprogramado* | *Sistema multiprogramado* |
|   -                      |   -                       |
|Un programa puede o no ingresar a una única partición de memoria | Múltiples programas comparten diversas particiones de memoria. Pueden ser particiones de tamaño fijo o particiones de tamaño variable|
|![Sistema monoprogramado](./img/Sistema_monoprogramado.png) |![Sistema multiprogramado](./img/Sistema_multiprogramado.png)|

### Requisitos del administrador de memoria

1) Reubicación: Permitir el recalculo de direcciones de memoria de un proceso reubicado.
2) Protección: Evitar el acceso a posiciones de memoria sin el permiso expreso (no direcciones absolutas)
3) Compartición: Permitir a procesos diferentes acceder a la misma porción de memoria
4) Organización Lógica: Permitir que los programas se escriban como módulos compilables y ejecutables por separado.
5) Organización Física: Permitir el intercambio de datos en la memoria primaria y secundaria.

### Estrategias

Están dirigidas a la obtención del mejor uso del recurso memoria principal, estás pueden ser:

1) Estrategia de solicitud (búsqueda)
(Cuando obtener un fragmento de programa)

   - Estrategias de búsqueda por demanda.
   - Estrategias de búsqueda anticipada.

2) Estrategia de ubicación
(Donde se colocará (cargar) un fragmento de programa nuevo)

3) Estrategia de reposición.
(Qué fragmento de programa descarga, para cargar uno nuevo)

#### Técnicas para administrar memoria

##### 1. Partición Fija

La memoria principal se divide en un conjunto de particiones de tamaño fijo durante el inicio del sistema.
Un proceso se puede cargar completamente en un partición de tamaño menor o igual.

- Ventajas:
  - Sencilla de implementar
  - Poca sobrecarga al SO
- Desventaja:
  - Fragmentación interna
  - Nro. fijo de procesos activos

###### Estrategias de la partición fija

- Solicitud.
  - Por demanda
- Ubicación.
  - Partición de igual tamaño: Si el proceso cabe en una partición se puede cargar
  - Partición de diferente tamaño: Asignar a la partición más pequeña
- Reemplazo.
  - Uno de los procesos se saca, según el planificador.


|Particiones del mismo tamaño|Particiones de distinto tamaño|
|-|-|
|![Particiones de mismo tamaño](./img/PF_Estrategias_de_Ubicación_1.png)|![Particiones de distinto tamaño](./img/PF_Estrategias_de_Ubicación_2.png)|

Si un programa no cabe en una partición, el programador debe diseñarlo en módulos cargables.

El uso de memoria es muy ineficiente, no importa el tamaño del proceso, ocupara toda la partición, se genera fragmentación interna.

![Fragmentación interna](./img/PF_Fragmentación_interna.png)

##### 2. Partición Dinámica

Las particiones se crean dinámicamente por demanda. Son variables en tamaño y número.
Cada proceso se carga completamente una única partición del tamaño del proceso.

- Ventajas:
  - No existe fragmentación interna.
- Desventajas:
  - Fragmentación externa
  - Se debe compactar la memoria
  - El compactado toma tiempo

El uso de memoria es muy ineficiente, se generan muchos huevos entre las particiones, cada vez más pequeñas. Se genera la fragmentación externa.

Cada cierto tiempo se debe compactar los segmentos libres, para que estén contiguos.

![Fragmentación externa y compactación](./img/PD_Fragmentación_externa_y_compactación.png)

###### Estrategias de la partición dinámica

- Solicitud:
  - Por demanda
- Ubicación
  - *Primer ajuste*: El primer bloque disponible (parte del inicio)
    - Es bueno, con baja compactación. Puebla el inicio de la memoria
  ![Primer ajuste](./img/PD_Primer_Ajuste.png)
  - *Siguiente ajuste*: El siguiente bloque disponible que ubique (parte desde la ubicación actual)
    - Puebla el final de la memoria, el siguiente bloque libre siempre está al final de la memoria.
![Siguiente ajuste](./img/PD_Siguiente_Ajuste.png)
  - *Mejor ajuste*: El bloque disponible que deje el menor espacio libre (búsqueda exhaustiva)
    - Tiene peores resultados, dado que busca la partición que deje el hueco más pequeño, la memoria se llena de huecos pequeños. Se compacta con más frecuencia
![Mejor ajuste](./img/PD_Mejor_Ajuste.png)
  - *Peor ajuste*: Se asigna el hueco **más grande**, hay que buscar en la lista completa de huecos (salvo si está ordenada por tamaño)
- Reemplazo
  - Uno de los procesos se saca, según el planificador.

##### 3. Paginación Simple

</span>
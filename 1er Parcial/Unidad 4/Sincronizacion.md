# Concurrencia y sincronización

---

**Distintas formas de concurrencia**:

- *Multiprogramación*: implica múltiples procesos activos en memoria en un momento dado. Los procesos se van turnando en el uso de los recursos. Si bien hay varios procesos en memoria, solo uno puede ser ejecutado a la vez en una CPU
- *Multiprocesamiento*: manejo de múltiples procesos en un sistema multiprocesador, Muchos procesos ejecutando al mismo tiempo.
- *Procesamiento distribuido*: manejo de múltiples procesos en múltiples computadores distribuidas para ejecutar.
- *Compartición de recursos y competición por los recursos*:

**¿Por qué se usa la concurrencia?**

- Mejora la performance. Debido a la multiprogramación, si un proceso se encuentra bloqueado, para no dejar a la CPU ociosa, necesito ejecutar otro proceso en ese instante.
- Aplicaciones estructuradas: por cuestiones de diseño, un programa puede ser definido como un conjunto de procesos/hilos concurrentes.

## Condición de carrera

Situación donde varios actores (hilos o procesos) modifican datos compartidos y se obtienen diferentes resultados finales dependiendo del orden en el que se ejecuten los procesos o hilos (se dice que depende de la "velocidad" de ejecución de ellos, de ahí el nombre).
Para garantizar la coherencia debemos asegurar que solo uno de los procesos pueda acceder a manipulación de datos a la vez.Para esto hay que sincronizarlos. Solamente vamos a sincronizar dos procesos cuando acceden a los mismo datos (ya sea que los dos quieran escribir o uno de ellos quiere escribir y el otro leer). Pero, si los dos acceden en modo lectura, no hace falta sincronizarlos.
La sección donde puede ocurrir la condición de carrera se conoce como *sección crítica*

## Formas de interacción entre procesos

- Comunicación entre procesos
- Competencia de los procesos por los recursos.
- Cooperación de los procesos vía compartición
- Cooperación de los procesos vía comunicación

Si hay varios procesos, los mismo van a competir por el uso de memoria y demás recursos. En la competencia, el SO se va a encargar de decidir qué proceso recibe qué recursos y por cuánto tiempo.

![Interacción entre procesos](img/Interaccion%20entre%20procesos.png)

### Requisitos que deben cumplirse dentro de la sección crítica

**Mutua exclusión**: sólo un proceso puede estar en la sección crítica usando un recurso. No puede haber ningún otro proceso o hilo en la misma sección para usar ese mismo recurso.

**Progreso**: cuando un proceso termina de usar el recurso de la sección crítica debe dar a aviso así los demás hilos o procesos pueden acceder a él

**Espera limitada**: la espera para entrar a la región crítica debe ser limitada. Se relaciona con el progreso. El proceso que está ejecutando en la región crítica debe avisar cuando termine de usarla porque sino los demás procesos que están esperando nunca se enterarían que está libre.

**Velocidad relativa**: qué tan rápido va a ejecutar un proceso sus instrucciones. Nunca puede ser predicho porque puede ocurrir interrupciones en el medio. Por más mínima que sea la operación no podemos decir con certeza cuánto va a durar.

*Consideraciones*:

- La sección critica debe ser lo más pequeña posible. Si la sección critica fuera muy grande (al punto en que un proceso debería esperar a que finalice otro para poder ejecutar) entonces no se podría tener procesos concurrentes porque la sección permanecería bloqueada.
- Un proceso que no está en una sección critica no interfiere de ninguna manera con otros procesos, entonces puede ejecutar lo que quiera.
- La permanencia en la sección critica debe ser por un tiempo finito y reducido
- Un proceso puede tener muchas secciones críticas.

**Condiciones de Bernstein**: Si se cumplen las siguiente condiciones entonces no existe la posibilidad de condición de carrera y no estamos en presencia de una sección critica.

- $R(A) \cap W(B) = {\empty}$ : Las variables que va a leer el proceso A no deben ser las que va a escribir el proceso B
- $R(B) \cap W(A) = {\empty}$ : igual a lo anterior
- $W(A) \cap W(B) = {\empty}$ : El conjunto de escritura de A debe ser distinto al conjunto de escritura de B. No deben escribir sobre las mismas variables.

### Posibles soluciones para la condición de carrera/Sección critica
49
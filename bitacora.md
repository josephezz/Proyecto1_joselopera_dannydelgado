# Protecto corte 1

# Integrantes:

* Jose Alejandro Lopera Coaji
* Danny Santiago Delgado Peralta

## Punto 5 — Caso propio con tres colas de prioridad

Para este punto se construyó un caso propio utilizando tres colas de prioridad, cada una con un algoritmo y quantum diferente. La cola 1 utiliza Round Robin (RR) con quantum 2, la cola 2 utiliza SJF con quantum 3 y la cola 3 utiliza FIFO con quantum 1. Los procesos A, B, C, D, E y F fueron distribuidos entre estas tres colas de acuerdo con su prioridad.

El objetivo de este caso fue observar cómo cambia la planificación cuando los procesos se encuentran distribuidos en diferentes colas y cada una utiliza una estrategia diferente. De acuerdo con el funcionamiento del simulador, las colas tienen sus propios algoritmos y quantum y son recorridas de forma circular, lo que influye en el orden en que los procesos reciben la CPU.

Al ejecutar el caso con tres colas se obtuvo un tiempo total de simulación de **23 unidades de tiempo**, un tiempo total de espera de **64 unidades** y un tiempo promedio de espera de **10.667 unidades**.

La secuencia de ejecución obtenida fue:

A (2) D (2) E (1) B (2) C (3) E (1) A (2) C (1) E (1) B (1) E (1) A (1) E (1) E (1) F (1) F (1) F (1)

Además, el simulador generó el diagrama de Gantt correspondiente en el archivo `test/mi_caso_prioridades.png`, donde se puede observar gráficamente el orden y la duración de las ejecuciones de cada proceso.

Posteriormente, se realizó una segunda prueba utilizando exactamente los mismos procesos, conservando sus tiempos de llegada y sus tiempos de ejecución, pero colocándolos todos en una sola cola. En esta segunda configuración se utilizó Round Robin con quantum 2 para todos los procesos.

El resultado de esta segunda prueba fue un tiempo total de simulación de **23 unidades de tiempo**, un tiempo total de espera de **76 unidades** y un tiempo promedio de espera de **12.667 unidades**.

La secuencia de ejecución en el caso de una sola cola fue:

A (2) C (2) B (2) E (2) D (2) A (2) F (2) C (2) B (1) E (2) A (1) F (1) E (2)

También se generó un segundo diagrama de Gantt en `test/mi_caso_una_cola.png`, que permite comparar visualmente la planificación con la obtenida utilizando las tres colas.

Al comparar ambos experimentos se observa que el tiempo total de simulación fue de **23 unidades** en los dos casos. Sin embargo, el tiempo de espera fue diferente: con tres colas se obtuvo un total de **64 unidades de espera**, mientras que con una sola cola se obtuvo un total de **76 unidades**. De la misma manera, el tiempo promedio de espera fue de **10.667 unidades** con tres colas y de **12.667 unidades** con una sola cola.

La diferencia se debe a que, en el primer caso, los procesos están distribuidos entre diferentes niveles de prioridad y cada cola utiliza un algoritmo y quantum diferente. Esto hace que los procesos reciban la CPU siguiendo una secuencia determinada por la interacción entre las prioridades, los algoritmos y los tiempos de cada cola. En cambio, cuando todos los procesos se colocan en una sola cola, todos son planificados utilizando la misma estrategia Round Robin con quantum 2.

La diferencia también puede observarse en las secuencias de ejecución. Aunque los procesos utilizados son los mismos en ambos experimentos, el orden en que reciben la CPU cambia considerablemente. Esto demuestra que la distribución de los procesos en diferentes colas de prioridad puede modificar el comportamiento del planificador y los tiempos de espera.

En conclusión, el experimento permitió comprobar el efecto que tiene utilizar múltiples colas de prioridad frente a utilizar una sola cola. Para este caso particular, ambas configuraciones terminaron la simulación en 23 unidades de tiempo, pero la configuración con tres colas produjo un menor tiempo total y promedio de espera. Por lo tanto, la utilización de diferentes algoritmos y quantums para las distintas colas modificó la forma en que los procesos fueron atendidos y la cantidad de tiempo que permanecieron esperando por la CPU.
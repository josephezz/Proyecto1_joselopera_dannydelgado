# Proyecto corte 1

# Integrantes:

* Jose Alejandro Lopera Coaji
* Danny Santiago Delgado Peralta

## Puntos 1 a 4 — Implementación de los algoritmos de planificación

### Punto 1 — El proceso que llega a una cola (`procesar_llegadas`)

```cpp
if (c.estrategia == Estrategia::SRT || c.estrategia == Estrategia::SJF) {
    insertar_por_restante(c.listos, p);
} else {
    c.listos.push_back(p);
}
```

Esta es la línea que separa a los cuatro algoritmos en dos grupos. En `FIFO` y `RR` el orden de
la cola de listos lo decide únicamente el orden de llegada, por lo que un proceso que llega
siempre se agrega al final (`push_back`). En `SJF` y `SRT`, en cambio, el orden lo decide la
ráfaga: como un proceso recién llegado todavía no ha recibido nada de CPU, su `restante` es
igual a su `ejecucion`, así que insertarlo "por tiempo restante" en este instante equivale a
insertarlo por tamaño de ráfaga, que es justamente el criterio de SJF y de SRT.

### Punto 2 — La inserción ordenada (`insertar_por_restante`)

```cpp
void insertar_por_restante(std::deque<Proceso *> &cola, Proceso *p) {
    for (auto it = cola.begin(); it != cola.end(); ++it){
        if (p->restante < (*it)->restante){
            cola.insert(it, p);
            return;
        }
    }
    cola.push_back(p);
}
```

La condición `p->restante < (*it)->restante` (con `<` estricto, no `<=`) es la que garantiza el
desempate pedido por el enunciado: un proceso solo se inserta delante de otro si su tiempo
restante es **estrictamente menor**. Si dos procesos tienen el mismo restante, la condición es
falsa y el recorrido continúa, de modo que el proceso nuevo termina detrás del que ya estaba.
Así, el desempate lo decide el orden de llegada, porque el que ya estaba en la cola llegó antes.

### Punto 3 — El proceso que no ha terminado (`switch` dentro de `planificar`)

```cpp
switch (cola.estrategia) {
    case Estrategia::FIFO:
        cola.listos.push_front(actual);   // ya resuelto, sirve de referencia
        break;
    case Estrategia::RR:
        cola.listos.push_back(actual);
        break;
    case Estrategia::SJF:
        cola.listos.push_front(actual);
        break;
    case Estrategia::SRT:
        insertar_por_restante(cola.listos, actual);
        break;
}
```

Aquí está la diferencia más visible entre los cuatro algoritmos, y es literalmente una palabra
distinta en cada caso:

- **RR** usa `push_back`: el proceso que agota su quantum sin terminar pierde su lugar y se va
  al final de la cola, dando paso a los demás. Esta es la esencia de Round Robin.
- **SJF** usa `push_front`, igual que FIFO: como SJF no es expropiativo, el proceso conserva la
  CPU, es decir, sigue siendo el primero en la cola la próxima vez que le corresponde el turno.
- **SRT** usa `insertar_por_restante`: el proceso vuelve a la cola, pero no al frente ni al
  final, sino en la posición que le corresponde según su nuevo `restante`, que ya quedó
  actualizado por `sumar_cpu` antes de este `switch`.

### Punto 4 — La expropiación de SRT

```cpp
if (cola.estrategia == Estrategia::SRT) {
    for (Proceso *p : cola.llegada) {
        int tiempo_hasta_llegada = p->llegada - ahora;
        if (tiempo_hasta_llegada >= 0 &&
            tiempo_hasta_llegada < quantum_asignado &&
            p->ejecucion < actual->restante - tiempo_hasta_llegada) {
            quantum_asignado = tiempo_hasta_llegada;
            cambiar_de_cola = false;
            break;
        }
    }
}
```

Esta es la única parte del código que hace que SRT sea expropiativo. Se revisan los procesos que
todavía no han llegado a la cola (`cola.llegada`, que está ordenada por tiempo de llegada) y, para
cada uno, se calcula `actual->restante - tiempo_hasta_llegada`: eso es lo que le **quedaría** al
proceso en ejecución en el instante en que ese proceso nuevo llegue, si no se le interrumpiera.
Si la ráfaga del que llega (`p->ejecucion`) es menor que ese valor, se produce la expropiación:

- `quantum_asignado = tiempo_hasta_llegada` corta el turno justo en el instante de la llegada, en
  vez de agotar el quantum completo.
- `cambiar_de_cola = false` evita que, al terminar este turno recortado, la simulación pase a la
  siguiente cola de prioridad; la misma cola vuelve a jugar de inmediato, y como el proceso recién
  llegado queda con menos restante, será él quien reciba la CPU en la siguiente iteración.

Se usa `<` estricto y no `<=` a propósito: si el que llega tiene exactamente el mismo restante que
le quedaría al actual, no vale la pena expropiar (evita un cambio de contexto innecesario que no
mejoraría el tiempo de espera).

## Comprobación con el caso del taller

Los cuatro casos usan los mismos procesos: P1 (ráfaga 7, llega en 0), P2 (ráfaga 4, llega en 2),
P3 (ráfaga 1, llega en 4) y P4 (ráfaga 4, llega en 5). RR y SRT usan quantum 2.

### FIFO — espera promedio 4.750

Secuencia: `P1(2) P1(2) P1(2) P1(1) P2(2) P2(2) P3(1) P4(2) P4(2)`

Aunque el turno de cada cola dura un quantum, `push_front` hace que P1 recupere el frente de la
cola cada vez que le toca de nuevo, así que en la práctica ejecuta sus 7 unidades sin que ningún
otro proceso se interponga. Solo cuando P1 termina (en t=7) entran P2, P3 y P4, siempre en el
orden en que llegaron. Nadie espera más de lo que le corresponde por haber llegado después de un
proceso más largo: por eso el promedio (4.750) es el más alto de los cuatro.

### SJF — espera promedio 4.000

Secuencia: `P1(2) P1(2) P1(2) P1(1) P3(1) P2(2) P2(2) P4(2) P4(2)`

P1 ya está en ejecución cuando llegan P2, P3 y P4, y SJF no expropia, así que P1 corre completo
igual que en FIFO. La diferencia aparece en t=7: en vez de atender a P2 (que llegó primero, en
t=2), la cola de listos —ordenada por `insertar_por_restante`— tiene al frente a P3, cuya ráfaga
(1) es menor que la de P2 (4). Por eso P3 se ejecuta antes que P2, aunque haya llegado después.
Ese único cambio de orden basta para bajar el promedio de 4.750 a 4.000.

### RR con quantum 2 — espera promedio 5.000

Secuencia: `P1(2) P2(2) P1(2) P3(1) P2(2) P4(2) P1(2) P4(2) P1(1)`

Aquí `push_back` obliga a P1 a ceder la CPU cada 2 unidades y esperar su turno detrás de los
demás. Por eso P1, que en FIFO y SJF terminaba en t=7, ahora termina en t=16: cada vez que agota
su quantum, entra un proceso distinto (P2, luego P3, luego P2 otra vez, etc.) antes de que le
vuelva a tocar. Ese reparto constante del procesador es lo que produce el promedio más alto de
espera (5.000): nadie monopoliza la CPU, pero todos —sobre todo P1— esperan más tiempo acumulado.

### SRT con quantum 2 — espera promedio 3.000

Secuencia: `P1(2) P2(2) P3(1) P2(2) P4(2) P4(2) P1(2) P1(2) P1(1)`

La diferencia frente a RR está en t=4: P3 llega con ráfaga 1, y en ese instante a P2 le quedan 2
unidades de ráfaga restante. Como 1 < 2, se cumple la condición de expropiación del Punto 4: P2
se interrumpe de inmediato (sin esperar a que acabe su quantum) y P3 recibe la CPU. Al ejecutarse
apenas llega, P3 nunca acumula tiempo de espera (columna `Espera = 0` en la tabla de resultados).
Ese es el efecto de "atender primero al que menos le falta" en cuanto aparece: es lo que hace que
SRT tenga, de los cuatro, el promedio de espera más bajo.

## Resumen comparativo

| Algoritmo | Línea que lo distingue | Espera promedio | Por qué |
| --- | --- | --- | --- |
| FIFO | `push_front(actual)` | 4.750 | Nunca se interrumpe; el orden de llegada manda de principio a fin |
| SJF | `push_front(actual)` + inserción por ráfaga en la llegada | 4.000 | No se interrumpe en ejecución, pero el orden de atención sí cambia por ráfaga |
| RR | `push_back(actual)` | 5.000 | Se interrumpe siempre al agotar el quantum, sin mirar ráfaga ni restante |
| SRT | `insertar_por_restante(actual)` + condición de expropiación | 3.000 | Se interrumpe también en medio del quantum si llega algo más corto |

La coincidencia de las cuatro cifras con la tabla del enunciado, junto con el hecho de que cada
algoritmo produce una secuencia de ejecución distinta (no solo un promedio distinto), confirma que
los cuatro puntos están resolviendo lo que el algoritmo correspondiente exige, y no solo
reproduciendo el comportamiento de FIFO por coincidencia.

---

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

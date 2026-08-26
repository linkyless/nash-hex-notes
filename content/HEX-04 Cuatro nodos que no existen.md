---
title: HEX-04 Cuatro nodos que no existen
tags:
  - hilo
---
Primera pieza de código real del proyecto, y la única donde parto con ventaja.

El tablero es una matriz de `5x5` con un `1` donde hay ficha de un jugador, un `-1` donde hay del otro y un `0` donde no hay nada. Colocar es asignar, y los movimientos legales son las casillas a cero. Nada de esto merece un párrafo y sin embargo aquí estamos.

Lo que sí lo merece es la forma del tablero, que confunde a todo el mundo la primera vez. A mí me tuvo un buen rato convencido de que necesitaba coordenadas raras o un sistema hexagonal de esos que salen en los artículos con diagramas de colores. Pues resulta que no.

## El rombo es una matriz

En memoria no hay nada hexagonal (que yo sepa). Es una lista de listas indexada por fila y columna, idéntica a cualquier cuadrícula. Lo único que cambia son dos cosas.

1. Cómo dibujarlo: cada fila va indentada un espacio más que la anterior y de ahí sale el rombo. Es puramente cosmético y si imprimiera las filas alineadas el juego funcionaría exactamente igual, solo que sería insoportable de mirar.

2. Cómo lidio con los vecinos: en una cuadrícula normal una casilla tiene cuatro vecinos, u ocho si cuentas diagonales, mientras que en Hex tiene seis. Son las cuatro ortogonales de siempre más dos diagonales concretas. Solo dos, ni una más. Mírese abajo los desplazamientos si estamos parados en la celda $(r, c)$:

$$D = \{(0,1),\ (0,-1),\ (1,0),\ (-1,0),\ (-1,1),\ (1,-1)\}$$
```python
# En board.py, la geometría entera del juego se reduce a estos dos arrays:
self.d_row = np.array([0,  0, -1, 1, -1,  1])
self.d_col = np.array([1, -1,  0, 0,  1, -1])
```

Ahí está toda la geometría del juego y no parece excesivamente difícil. Con esos seis desplazamientos, la detección de victoria, el árbol de búsqueda y la red neuronal trabajan sobre una matriz cuadrada corriente sin enterarse de que el tablero es hexagonal.

![[Neighbours-in-the-hexagonal-grid-of-areas-and-in-the-triangular-grid-of-nodes.png]]

Guarda ese conjunto en alguna parte de la cabeza, porque va a volver más adelante en un contexto donde ahora mismo no pinta absolutamente nada, y va a salvar un argumento entero. Está en [[LEM-forma canónica]] y no digo más porque me apetece que sorprenda.
## Detectar la victoria

La pregunta es si un jugador ha conectado sus dos bordes. La forma obvia consiste en lanzar una búsqueda desde cada ficha del borde superior y ver si alcanzas el inferior, lo cual funciona perfectamente y es correcto. Pero también es lento, porque el árbol va a llamar a esta función cientos de miles de veces por partida. Quizás no tanto, pero sigue siendo lento, joder. No me gustan las cosas lentas aquí. Como mucho tolero mi aprendizaje, pero poco más.

La forma buena usa [[LEM-union-find]] con un truco que en programación competitiva se ve bastante y que aquí encaja como si lo hubieran diseñado a propósito.

Además de las veinticinco casillas del tablero, añado **cuatro nodos que no corresponden a ninguna casilla**. Uno por borde. Norte, sur, este y oeste. No existen, no se pueden jugar y no se dibujan.

Cuando coloco una ficha la uno a sus vecinos del mismo color, y si además toca un borde la uno también al nodo virtual de ese borde. Entonces la pregunta "¿ha conectado blancas el norte con el sur?" se transforma en "¿están el nodo norte y el nodo sur en el mismo conjunto?", que es comparar dos raíces en el UFDS.

```python
self.north_cell = self.board_size ** 2
self.south_cell = self.board_size ** 2 + 1
self.east_cell  = self.board_size ** 2
self.west_cell  = self.board_size ** 2 + 1

# Y para conectar bordes:
if player == 1: # Blancas conectan Norte-Sur
    if row == 0:
        self.ufds_white.join(current_cell, self.north_cell)
    if row == self.board_size - 1:
        self.ufds_white.join(current_cell, self.south_cell)
```

Con compresión de caminos y unión por tamaño el coste amortizado por consulta es

$$O(\alpha(n))$$

donde $\alpha$ es la función inversa de Ackermann. Aquí es donde tocaría explicar la función de Ackermann, así que mejor lo dejamos en que para cualquier entrada que puedas escribir sin notación científica ese valor es menor que cinco o seis según recuerdo, y en la práctica la operación es constante. Si alguno quiere el detalle completo, existe y es una lectura entretenida durante unos diez minutos.

Hay un detalle de implementación que es que uso dos union-find separados, uno por jugador, porque las conexiones de blancas no tienen nada que ver con las de negras. Y como son estructuras distintas, los índices de sus nodos virtuales pueden repetirse sin colisionar, así que el nodo norte de blancas y el nodo este de negras son literalmente el mismo número. No pasa nada porque viven en universos separados. Es el tipo de cosa que no me importa.

## La interfaz

El detalle de diseño que más rendimiento me ha dado en todo el proyecto, y no tiene nada que ver con optimizar.

El árbol de búsqueda no puede saber nada de Hex. Le pide al tablero exactamente cuatro cosas, que son los *movimientos legales*, *aplicar un movimiento*, *si la partida ha terminado* y *quién ha ganado*.

``` python
choices = node.board.valid_choices()  # 1. Movimientos legales
new_board.place(row, col)             # 2. Aplicar movimiento
if node.board.winner is not None:     # 3. ¿Ha terminado y quién ha ganado?
    ...
```

Nada más. Y aquí lo importante está en que **NO puede tener nada más**, porque si el árbol conoce la geometría hexagonal, el día que algo falle no vas a saber si el fallo está en la búsqueda o en el juego. Tengo que ser capaz de separarlos por completo, y esto pensándolo en abstracto es bastante complicado si no estás acostumbrado a construir cosas grandes.

Suena a manía sacada de un manual de buenas prácticas, y lo es, hasta que llega el momento en el que el sistema tiene piezas que se retroalimentan, nada casca, la curva no sube y aislar un fallo pasa de ser una comodidad a ser lo único que te queda.

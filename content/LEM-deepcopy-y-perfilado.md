---
title: LEM-deepcopy y perfilado
tags:
  - lema
---
Primera versión funcional del self-play, `26s` para generar cinco partidas. Perfilando con `cProfile`, `17` de esos `26` estaban dentro de `copy.deepcopy`.

## Por qué es tan caro

`deepcopy` está hecho para copiar cualquier cosa sin saber nada de ella. Recorre el objeto entero de forma recursiva y va manteniendo un diccionario para no duplicar dos veces la misma referencia y para detectar ciclos. Para cada trozo pregunta si tiene método propio de copia, si no lo tiene prueba con el protocolo de serialización, y en última instancia reconstruye vía pickle.

Sobre un tablero de Hex eso significa copiar el array, las dos estructuras union-find, sus listas internas y un par de enteros sueltos, cada uno con su entrada en el diccionario.

Paga el precio de ser genérica.

## La alternativa

Un método `copy` escrito a mano que sabe exactamente qué hay dentro.

```python
def copy(self):
    new = HexBoard.__new__(HexBoard)
    new.board = self.board.copy()
    new.turn = self.turn
    new.winner = self.winner
    new.ufds = [u.copy() for u in self.ufds]
    return new
```

## Lo que aprendí de aquí

Mi predicción antes de perfilar era que el tiempo estaba en las pasadas de la red neuronal, porque es la operación que suena pesada. Estaba equivocado por un factor considerable.

El árbol crea una copia del tablero por cada nodo que expande. Con doscientas simulaciones y veinte movimientos son unas cuatro mil copias por partida frente a doscientas pasadas de red.

La intuición sobre dónde está el cuello de botella tiende a fijarse en la operación conceptualmente más gorda, no en la más frecuente. Y la más frecuente casi siempre gana.

## La decisión de fondo

Guardar el tablero en cada nodo, o guardar solo el movimiento y reconstruir la posición bajando desde la raíz cuando haga falta.

La primera gasta memoria y evita recomputar. La segunda ahorra memoria y te obliga a replicar la partida en cada consulta. En `5x5` la memoria no es problema, así que elegí la primera y pagué con esta tarde de perfilado. Con tableros grandes la comparación cambia.

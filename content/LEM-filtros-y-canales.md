---
title: LEM-filtros y canales
tags:
  - lema
---
Un filtro es una matriz pequeña de pesos que se desliza por toda la entrada. En cada posición multiplica lo que ve por sus pesos, suma, y escribe un número. El resultado es un mapa del tamaño de la entrada.

Ese mapa es un **canal**. Con `16` filtros salen `16` canales, o sea `16` mapas apilados.

## Lo que cambia en la segunda capa

Un filtro de la primera capa mira la entrada cruda, que en Hex es un solo mapa con unos, menos unos y ceros. Puede detectar cosas simples, del tipo "ficha propia aislada" o "zona vacía".

Un filtro de la segunda capa recibe los `16` canales de la primera, así que ya no es una cuadrícula plana sino un bloque de `16x3x3`. Al atravesarlos todos a la vez, **cruza** los patrones simples y puede activarse ante combinaciones. Ahí es donde aparecen cosas del tipo "ficha propia junto a hueco vecino", que empieza a parecerse a un concepto táctico.

Esa es toda la profundidad. Cada capa combina lo que encontró la anterior.

Conviene no tomarse los nombres muy en serio. Los patrones que detecta una red no tienen ninguna obligación de parecerse a conceptos que un humano haya inventado para hablar del juego, y probablemente la mayoría no se parezcan a nada.

## Parámetros

$$C_{\text{out}} \cdot (C_{\text{in}} \cdot k^2 + 1)$$

No depende del tamaño del tablero. Los mismos pesos se aplican en todas las posiciones, que es lo que distingue una convolución de una capa densa sobre la imagen aplanada.

## Traslaciones

Un patrón detectado en una esquina se detecta igual en la otra, porque el filtro es el mismo. No hace falta un ejemplo de entrenamiento por cada sitio donde pueda aparecer.

Con rotaciones y reflexiones no pasa. Esas hay que meterlas a mano duplicando los datos, cosa que en Hex se hace con la rotación de `180` grados de [[LEM-forma canónica]].

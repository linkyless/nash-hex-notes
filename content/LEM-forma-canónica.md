---
title: LEM-forma canónica
tags:
  - lema
---

$$\phi(B) = (-B)^{\top}$$

Se aplica cuando mueve el segundo jugador. La negación pone sus fichas como propias, la trasposición convierte su objetivo en el del primero.

La negación no tiene misterio. La trasposición sí, porque mueve las fichas de sitio y hay que comprobar que después el juego sigue siendo el mismo.

## La vecindad aguanta

Los seis vecinos de Hex son los desplazamientos

$$D = \{(0,1),\,(0,-1),\,(1,0),\,(-1,0),\,(-1,1),\,(1,-1)\}$$

Trasponer manda $(r,c)$ a $(c,r)$, con lo cual a un desplazamiento $(dr,dc)$ le toca $(dc,dr)$. Aplicándolo a la lista, los cuatro ortogonales se intercambian entre sí por parejas y las dos diagonales se intercambian entre ellas.

Sale el mismo conjunto. Dos casillas son vecinas antes si y solo si lo son después, o sea que el grafo es literalmente el mismo.

## Los bordes también

La fila `0` va a la columna `0` y la fila `n-1` a la columna `n-1`, con lo cual norte pasa a oeste y sur pasa a este. Como además hemos negado, el objetivo del segundo jugador se ha convertido en el del primero. Hecho.

## Y esto no es casualidad

Podría haber salido mal. Si el conjunto de vecinos no fuera invariante, la forma canónica sería una aproximación sucia que funciona más o menos.

Sale bien porque la trasposición **es** la simetría del rombo que intercambia los papeles de los dos jugadores. No se está usando una coincidencia numérica, se está usando la estructura del juego. En ajedrez no existe nada parecido, porque los peones solo avanzan en una dirección y la simetría se rompe.

## La otra simetría

Girar `180` grados manda $(r,c)$ a $(n-1-r, n-1-c)$, o sea que niega cada desplazamiento, y la lista es cerrada bajo negación. Norte y sur se intercambian, este y oeste también, y **ningún jugador cambia de objetivo**, que es la diferencia con la trasposición.

Por eso esta no sirve para canonizar y sí para duplicar los datos de entrenamiento, rotando el tablero y el $\pi$ igual y dejando el value intacto.

## Lo que le pasa a la salida

Si la red mira $\phi(B)$, la policy que devuelve está indexada sobre $\phi(B)$. Para usarla hay que deshacerlo, reformando a `5x5`, trasponiendo y aplanando.

La lección general, que va más allá de este proyecto, es que cuando transformas la entrada de un modelo tienes que preguntarte en qué espacio vive la salida. Casi siempre en el transformado, y casi nunca donde tú la vas a leer. El fallo está en [[HEX-10 Algo anda mal]].

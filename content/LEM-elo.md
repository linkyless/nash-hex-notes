---
title: LEM-elo
tags:
  - lema
---
Elo es un modelo.

El modelo dice que a cada jugador le corresponde un número y que la probabilidad de que $A$ le gane a $B$ depende **solo** de la diferencia entre sus números.

$$P(A \text{ gana}) = \frac{1}{1 + 10^{(R_B - R_A)/400}}$$

El `400` es una convención elegida para que doscientos puntos de diferencia den alrededor de un `76%`. Podría haber sido otro número y la escala entera se estiraría.

## Lo de la transitividad

De esa forma sale la transitividad como consecuencia, no como hipótesis extra. Si $A$ gana a $B$ y $B$ a $C$, el modelo obliga a que $A$ gane a $C$.

El problema es que los juegos no siempre funcionan así. Puede haber un trío donde $A$ gana a $B$, $B$ gana a $C$ y $C$ gana a $A$, como en piedra papel tijera, y cuando eso pasa ningún número por jugador puede describirlo. No es que se calcule mal, es que el fenómeno no cabe en un escalar.

Con motores esto es real y no una curiosidad, porque cada uno tiene sus puntos ciegos y otro puede estar especializado en explotarlos.

## El error de encadenar

Medir el progreso comparando checkpoints adyacentes y sumando las diferencias acumula error en una sola dirección. Cada eslabón tiene ruido y todos apuntan hacia arriba por construcción, así que el ruido no se cancela, se suma.

Aquí eso dio `+1131` encadenando frente a `+610` midiendo directamente los extremos. Es medir una habitación poniendo un pie delante del otro.

## Cómo se hace bien

No encadenando nada. Se cogen todas las partidas del torneo y se ajustan todos los ratings a la vez, buscando los que hacen más verosímiles los resultados observados. Por dentro es una regresión logística donde el parámetro de cada jugador es su rating.

Tres cosas que hacen falta.

**Un ancla**, porque el ajuste solo depende de diferencias y sumarle mil a todo el mundo da lo mismo. Se fija un jugador en cero.

**Un prior**, porque un jugador invicto se va a infinito alegremente si nada lo ata. Un gaussiano débil basta y no distorsiona donde hay datos.

**Un parámetro para la ventaja de salida**, que en Hex es enorme. Si no está, esa ventaja se reparte entre los ratings y los contamina.

De propina, la matriz de segundas derivadas en el óptimo da las barras de error. Con cincuenta partidas por emparejamiento salen del orden de `±50`, o sea que dos motores separados por sesenta puntos no son distinguibles. Eso es una afirmación sobre el tamaño de la muestra y no sobre los motores, y decirlo es lo único que separa un número de un número honesto.

## Ventaja

Si hay ciclos, el modelo no los puede explicar y eso se nota en el ajuste. Con lo cual la transitividad deja de asumirse a ciegas y pasa a comprobarse cada vez.

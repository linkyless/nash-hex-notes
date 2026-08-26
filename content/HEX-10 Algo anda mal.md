---
title: HEX-10 Algo anda mal
tags:
  - hilo
---
![[what_happened.jpg]]

`40%`.

Ese es el número. La red entrenada durante dos horas con 30 iteraciones, pérdida bajando de `3.2` a `2.0` como en los libros, enfrentada contra una red **recién inicializada con pesos aleatorios**, y gana el cuarenta por ciento de las partidas. QUÉ COJONES.

O sea que entrenar la había empeorado. Treinta iteraciones para llegar a jugar peor que el ruido del principio. ¡Vamos! Si es que esto es increíble. Mi madre me pregunta por qué estoy tan feliz, seguro que a ella nunca le ha pasado esto.

Esta nota es lo que hice a partir de ahí, y va a estar organizada como lo estuvo en su momento, que es una lista de sospechosos a los que ir descartando. Al final queda uno. Spoiler, ese uno tampoco arregla el problema, pero eso es la nota siguiente.

## Antes de nada, por qué esto es distinto

En competitiva siempre hay un veredicto, bien sea WA, TLE o RTE.

Aquí el programa corre y todo está bien, los tensores tienen las formas correctas y todo está bien. No hay ni una excepción en dos horas de ejecución. Y hala, el sistema es una mierda y está roto. Lo único que te dice que algo va mal es un número al final que no es el que debería ser, y ese número no viene con un stack trace. Podría estar en la búsqueda, en la red, en la generación de datos, en la pérdida, en el bucle de entrenamiento, o en dos sitios a la vez. Va a ser en dos sitios a la vez, por cierto, porque aparentemente un bug se sentía demasiado solo. Pero eso yo todavía no lo sé.

## Sospechoso 1

**Hipótesis.** El instrumento miente. Mi arena está sesgada hacia una de las dos posiciones y el `40%` no significa nada. Esto va primero por una razón puramente práctica, y es que si el instrumento está roto todo el trabajo posterior es tiempo tirado. Es lo mismo que calibrar la báscula antes de discutir sobre la dieta. La única diferencia es que una báscula mal calibrada normalmente no aprende de tus errores y luego vuelve a pesarte convencida de que tiene razón.

**Prueba.** Cambio el orden de los argumentos, poniendo la red vieja donde iba la nueva.
**Resultado.** El resultado se da la vuelta limpiamente.
**Veredicto.** Descartado. La arena es justa.

## Sospechoso 2

**Hipótesis.** El entrenamiento va bien un rato y luego se estropea. Sobreajuste acumulado, o la red persiguiendo su propia distribución hasta caer en un pozo del que no sale. Es la explicación que uno espera cuando algo relacionado con aprendizaje va mal.
**Prueba.** Enfrento la iteración `10` contra la `0`, que es un punto mucho más temprano.
**Resultado.** `40%` otra vez, más o menos lo mismo.
**Veredicto.** Descartado, y esto está guay porque no hay un pico seguido de una caída, está mal **desde la primera iteración**. Eso elimina de golpe todas las hipótesis que requieren acumulación, y además convierte un fallo que tardaba dos horas en reproducirse en uno que tarda cuatro minutos. Esa fue una buena noticia porque, bien, por fin... por fin había conseguido empeorar el motor y además hacer que fuese más fácil demostrar que lo había empeorado.

## Sospechoso 3

**Hipótesis.** La red produce la misma policy para cualquier tablero, o sea que ha aprendido a ignorar la entrada.
**Prueba.** Le paso cuatro posiciones muy distintas entre sí y comparo las salidas.
**Resultado.** Las salidas son distintas. La policy sí depende de la posición.
Y aquí es donde salta el dato bueno, que no venía a buscar. Miro los números concretos en una posición donde había una **victoria en un movimiento**, y resulta que la casilla ganadora tiene un prior de `0.056` mientras que una casilla que no hace absolutamente nada tiene `0.086`.
**Veredicto.** Descartado el colapso, y apuntado algo peor. La red no ha aprendido nada útil, ha aprendido ruido correlacionado con el tablero. El problema era que sabía cosas falsas con suficiente **consistencia** como para que MCTS las tomase en serio.

Y eso es peor que no haber aprendido, porque el ruido aleatorio la búsqueda lo promedia y el ruido sistemático no. Uno se cancela y el otro se acumula.

![[wrong_positions.jpg]]

## Sospechoso 4

**Hipótesis.** Si la policy está mal, quizá el value también.
**Prueba.** Tres posiciones de resultado conocido.
**Resultado.** `+0.57` en una ganada. `-0.68` en una perdida. `+0.88` en otra ganada.
**Veredicto.** Impecable. El value funciona. Parece perfecto. Por primera vez en un rato hay algo que no está yéndose al carré. Decido disfrutar de ello durante aproximadamente treinta segundos.

O eso parece.

## Queda uno

Value bien, policy mal, desde la primera iteración, con ruido que depende de la posición. Eso apunta a que en algún sitio hay una correspondencia rota entre lo que la red dice y lo que yo leo. Y está en la forma canónica.

La forma canónica traspone el tablero cuando mueven negras, cosa que conté en [[HEX-07 Dos cabezas]] y cuya legitimidad está demostrada en [[LEM-forma canónica]]. Lo demostré. Estaba orgulloso de haberlo demostrado.

Lo que no se me pasó por la cabeza ni un segundo es qué le pasa a la **salida**. Raro no. Los bugs son bugs siempre.

Piénsalo despacio. La red mira un tablero traspuesto. Devuelve `25` logits. Esos logits están indexados sobre el tablero **que la red ha visto**, o sea el traspuesto. El logit número $k$ corresponde a la casilla $k$ del tablero traspuesto, no del real. Y yo los estaba leyendo con índices sacados de `valid_choices()`, que son índices del tablero real. O sea que en la mitad de cada partida, exactamente en los turnos de negras, los priors estaban permutados, la red aprendía un mapeo correcto para un jugador y otro barajado para el otro. En otras palabras, la red estaba jugando una partida y yo estaba leyendo sus respuestas como si estuviera mirando otro tablero. En términos técnicos como dije antes esto es una permutación. En términos humanos, llevaba media partida preguntándole a alguien "¿qué casilla?" y colocando la ficha una casilla al lado.

Y en el self-play pasaba lo mismo en espejo, porque guardaba el tablero traspuesto junto a un $\pi$ sin trasponer. Estaba entrenando con pares incoherentes la mitad de las veces, asociando posiciones con jugadas que no tenían nada que ver, con una consistencia admirable.

## El arreglo, y el chasco

Devolver la policy al marco real antes de usarla. Se reforma el vector a `5x5`, se traspone, se vuelve a aplanar. Tres operaciones encadenadas en una línea, que es lo bonito de programar en Python a veces.
```python
if black_moves:
    policy_logits = policy_logits.reshape(5, 5).T.reshape(-1)
```
Y en el self-play lo mismo pero al revés, trasponiendo el $\pi$ antes de guardarlo. Con cuidado de no joder el $\pi$ original, porque ese se usa después para muestrear el movimiento y necesita estar en coordenadas reales, o acabas colocando fichas en sitios que no elegiste. Casi lo hago, y habría sido un bug precioso, porque el motor habría seguido jugando partidas perfectamente legales.

```python
if black_moves:
    temp_pi = pi.reshape(5, 5).T.reshape(-1)
examples.append((Engine.transform_to_canonic_form(), temp_pi))
```

Vuelvo a medir esperando que esté resuelto yyyy...

`43%`.

Tres puntos de más. Estadísticamente una mejora, pero psicológicamente, una burla. Qué guapo. Al parecer hay más bugs, jaskjakjs.

Queda otro pibe, a ver si lo encontramos.
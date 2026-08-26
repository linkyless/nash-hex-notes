---
title: HEX-12 Saber si esto ha servido de algo
tags:
  - hilo
---
## Tres formas de medir mal dos redes

Enfrentar dos motores suena trivial y tiene tres maneras de salir mal. Me las comí las tres, una detrás de otra, y las cuento porque cada una enseña algo distinto y porque humillarme en público tiene un efecto pedagógico que ya quisieran algunos libros.

**1.** En Hex el primer jugador gana, cosa que estaba demostrada en [[LEM-robo de estrategia]] antes de que yo escribiera una sola línea de este proyecto. Si una red abre siempre, estás midiendo la ventaja de abrir. Se arregla alternando el color y no tiene más misterio, salvo el detalle de que yo lo había escrito en la tercera nota de este mismo cuaderno y aun así se me pasó. Escribir las cosas no sirve si luego no te lees.

**2.** El MCTS es determinista. Misma red, mismo tablero, misma partida. Siempre.

Mis primeras cien partidas fueron literalmente dos partidas distintas repetidas cincuenta veces cada una. El resultado salió `50/50` clavado. Una maravilla de experimento estadístico. Podía haber tirado una moneda y habría aprendido lo mismo.

Se arregla con **aperturas emparejadas**. Generas dos movimientos aleatorios, y esa misma apertura se juega dos veces con los colores intercambiados. Si la posición inicial favorecía a alguien, favorece a los dos por igual.

**3.** El Elo no es transitivo. Yo encadené comparaciones entre pares adyacentes, sumé las diferencias, y me salió `+1131`. La medición directa entre la primera red y la última decía `+610`.

Los dos números no pueden ser correctos y el bueno es el segundo, porque encadenar acumula error en una sola dirección. Es medir una habitación poniendo un pie delante del otro, porque cada paso tiene un sesgo minúsculo y 30 pasos después tu salón mide dos metros de más y tú estás muy convencido de tu medición. Está en [[LEM-elo]].

![[elo.png]]

Con las tres arregladas, las gráficas suben, los números cuadran, y yo me quedo bastante satisfecho conmigo mismo. Esa frase debería ir siempre acompañada de una música de tensión.

## Entonces me puse a jugar

Aquí es donde el proyecto se pone interesante de verdad y donde deja de parecerse a un ejercicio de clase.

Le dejé abrir a la máquina y fui rellenando la fila de abajo de izquierda a derecha. Eso no es una estrategia. Es la conexión este-oeste más tonta que existe, es lo que haría alguien a quien le acabas de explicar las reglas y que todavía no ha pensado absolutamente nada.

Gané. Volví a hacerlo, gané. 12 veces seguidas.

Nash nunca bloqueaba. Ni una vez en doce partidas, teniendo disponible una casilla que rompía mi cadena **y** extendía la suya al mismo tiempo, con cinco turnos de margen para jugarla. Cinco turnos.

Y lo más raro de todo, la red **sin entrenar** bloqueaba sin ningún problema.

O sea que treinta iteraciones de entrenamiento habían producido una red que perdía contra un truco que el ruido paraba sin despeinarse. Y todas mis métricas, todas, decían `97%`.

## Un dilema

Si mi motor gana el `97%` contra sus antepasados y pierde contra alguien que lleva diez minutos jugando a Hex, ¿es bueno o es malo?

Las dos afirmaciones son ciertas a la vez, están medidas, y no se contradicen. Miden cosas distintas y yo solo estaba mirando una.

Lo incómodo es que no hay respuesta obvia sobre cuál importa más. Si vas a publicar un número, el `97%` es el que impresiona. Si vas a poner el motor en una web para que juegue gente, el otro es el único que cuenta. Y como el primero es el que queda bien en LinkedIn, ya te puedes imaginar cuál publica la mayoría.

## El overlay

Menos mal que había construido la web (con ayuda de Claude) con un modo que pinta dónde ha gastado la búsqueda sus simulaciones.

Lo pedí por motivos puramente decorativos, porque me parecía bonito enseñar lo que el motor está pensando y porque quedaba bien en una captura. Y acabó siendo la única herramienta con la que podía haber diagnosticado esto. 

Pues puse la posición donde me faltaba **una** casilla para ganar y miré el mapa de calor.

La casilla que le salvaba la vida estaba completamente apagada. Cero visitas. La búsqueda ni la había mirado, ni de refilón.

Y la barra de evaluación llena hacia mi lado, o sea que el value **sabía** que iba perdiendo. La red no estaba ciega. Era la policy la que no le dejaba mirar dónde estaba el problema, que es una situación bastante deprimente si te paras a pensarla.

## La explicación

El término de exploración de PUCT es proporcional al prior. Con los números reales de esa posición, la casilla que la red adoraba tenía un prior alrededor de `0.4` y la casilla del bloqueo tenía `0.005`.

Un hijo ya visitado de los favoritos tiene $Q \approx 0.8$, porque Nash cree que va ganando y en ese momento tiene razón. La casilla del bloqueo, sin visitar, tiene $Q = 0$ y un término de exploración del orden de `0.05`.

```python
exploitation = self.mean_value() if self.visits > 0 else 0.0
exploration  = C_PARAM * self.prior * math.sqrt(self.parent.visits) / (1 + self.visits)
return exploitation + exploration
```

Se compara `0.8` contra `0.05` y gana el primero. Y se vuelve a comparar en la simulación siguiente. Y así.

**Nunca la visita.** Ni una fucking vez.

Y si la visitara **una sola vez** vería que pierdo en el turno siguiente, ese `-1` subiría por el árbol y hundiría el valor de todo lo demás. Una. Con una bastaba.

Es exactamente estudiar repasando solo los temas que ya te sabes. No es que suspendas el tema que no miraste, es que no te enteras de que ese tema existía y sales del examen pensando que ha ido bien, y luego te pasas dos semanas y media sin entender la nota.

Y explica también por qué la red virgen sí bloqueaba. Sus priors son todos parecidos, alrededor de `0.04`, así que reparte las visitas por todo el tablero y tropieza con el peligro sin buscarlo.

**La red virgen es peor jugando y a la vez inmune a este fallo, porque no tiene convicciones que la desvíen.**

Que es el precio literal de haber aprendido, escrito en una fórmula. Y si esto te suena a algo que ves fuera de los ordenadores, pues sí, a mí también. Dejo el paralelismo ahí sin desarrollarlo porque este es un cuaderno técnico y porque no me apetece que nadie se dé por aludido.

## Por qué nunca lo vio venir

Las partidas de entrenamiento las juegan dos copias de la misma red, con exactamente los mismos prejuicios.

Si la red decide en la iteración cinco que lo bueno es la diagonal central, las dos copias juegan la diagonal central y ninguna prueba jamás a rellenar una fila entera como haría un principiante. Con la temperatura en `8` movimientos solo hay azar en los primeros ocho, y solo dentro de lo que la propia policy considera plausible. Una casilla con prior `0.005` tampoco sale por muestreo, evidentemente.

Mil quinientas partidas y esa situación no apareció ni una sola vez. No es que la red la evaluara mal. Es que esa posición estaba fuera de su universo entero, como una habitación de tu casa que nunca has abierto porque la puerta está detrás de un armario y, con bastante honestidad, tampoco sabías que tenías esa habitación.

Y esto es exactamente lo mismo que decían mis gráficas desde el principio, con el value loss pegado a cero entre las iteraciones `12` y `20` y el mapa de aperturas concentrándose en una única casilla. Tenía la evidencia delante durante semanas y la leía como "converge bien", que es lo que una persona lee cuando quiere que algo converja bien, evidentemente.

## El parche

*Subir la constante de exploración no sirve de nada*, o eso pensé entonces, porque $c$ multiplica al prior y multiplicar `0.005` por tres sigue dejándolo en nada. Matemáticamente correcto. Prácticamente inútil. Hay pocas cosas más peligrosas que una frase que está bien.
 
Deja esa frase marcada con boli rojo en la pizarra. Es de las pocas de este cuaderno que están directamente equivocadas, y la corrijo dentro de dos notas.

Lo que hay que arreglar es el prior. Se mezcla la policy de la raíz con una distribución plana sobre los movimientos legales.

$$\tilde{P}(a) = (1-\lambda)\,P(a) + \frac{\lambda}{|A|}$$

```python
if node is self.root:
    uniform = 1.0 / len(choices)
    policy = policy * (1 - self.root_mix) + valid.float() * uniform * self.root_mix
```

Con `12` casillas libres y $\lambda = 0.25$, la del bloqueo pasa de `0.005` a unos `0.025`, o sea multiplicada por cinco. Y a la favorita, que estaba en `0.4`, le quitas un `0.08` y ni se entera.

**Ahí está todo el truco, en la asimetría.** Le pones un suelo a las despreciadas sin quitarle apenas nada a las buenas. La red sigue dirigiendo la búsqueda y a la vez ninguna casilla se queda a oscuras.

Esto es la versión determinista del **ruido Dirichlet** que AlphaZero mete en la raíz, y que estaba en el paper que leí, en la sección que evidentemente leí con toda la atención que merecía.

## El primer escalón

Fui subiendo el valor a ver cuándo bloqueaba.

Con $\lambda = 0.33$ se comía la derrota. Con $\lambda = 0.34$ bloqueaba.

Un salto de `0.01` decidiéndolo todo. Al principio me pareció gracioso y luego me pareció información, porque es un comportamiento de todo o nada, sin gradación ninguna. Cero visitas no cambian nada y una visita lo cambia todo.

Lo cual significa que cualquier valor calibrado contra un caso concreto está justo en el borde de un precipicio sin que tú lo sepas. Acabé en `0.4` por tener margen porque quería dormir tranquilo. Muy científico todo. Ese escalón vuelve a aparecer más adelante con otro número y en otro sitio. La segunda vez sí me molesté en calcular de dónde salía, básicamente porque ya me lo había comido una vez y hay cosas que solo se aprenden repetidas.

## Reentrenar

La mezcla parchea la búsqueda, sí, pero **la red sigue ciega y además bizca**.

Para que aprenda el patrón hay que reentrenar con la mezcla puesta durante el self-play, porque entonces las partidas dejan de ser todas la misma.

Reentrené con $\lambda = 0.4$ y la temperatura en `14`. Hora y media. La red nueva bloquea mi línea **con la mezcla apagada**, o sea que aprendió el patrón de verdad en vez de que la búsqueda la rescate en el último momento.

## Números

| enfrentamiento                        | partidas | resultado |
| ------------------------------------- | -------- | --------- |
| red nueva contra iteración `0`        | `100`    | `75%`     |
| red nueva contra la red vieja         | `100`    | `58%`     |
| iteración `20` contra la red vieja    | `100`    | `53%`     |
| mezcla activada contra mezcla apagada | `100`    | `56%`     |
|                                       |          |           |

La red vieja ganaba `97%` a una virgen. La nueva gana `75%`. **Y la nueva es mejor**, y esto no es una excusa mía, es la segunda fila de esa tabla. 

Lo que pasa es que la vieja había colapsado sobre un conjunto estrechísimo de líneas. Eso es devastador en un torneo controlado contra otro motor y a la vez deja un agujero que un humano encuentra en diez minutos con una estrategia que se le ocurriría a un niño.

Un `97%` medido solo contra ti mismo puede significar que eres muy bueno o que has aprendido a jugar muy bien contra tu propia ceguera. No hay forma de distinguirlo desde dentro, y esa frase es el resumen de todo lo que viene después.

![[loss.png]]

## Lo que descubrió sola

Y para terminar con algo que sí salió bien, porque si no esta nota se hace bastante cuesta arriba y yo tampoco quiero que la gente cierre la pestaña.

Mirando el mapa de aperturas iteración a iteración, la casilla sobre la que converge es el **centro** del tablero.

En Hex el centro es la casilla más fuerte que existe, porque está a la misma distancia de los cuatro bordes y participa en más conexiones potenciales que ninguna otra. En Hex serio, abrir en el centro se considera tan fuerte que es la jugada estándar que el rival roba con la regla del pastel.

Nadie le dijo eso. No hay ni una línea de código sobre el centro. Salió de mil quinientas partidas contra sí misma y de darse cuenta de que las posiciones donde controlaba esa casilla acababan ganando.

De todo lo que ha hecho el proyecto, eso es lo único que de verdad demuestra la tesis. Lo demás son porcentajes y yo ya he demostrado ampliamente que sé leerlos mal.

Pero esto aún no se había terminado. Y se ve en [[HEX-13 Doscientas veintiséis milésimas]].
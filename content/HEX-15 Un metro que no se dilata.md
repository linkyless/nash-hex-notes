---
title: HEX-15 Un metro que no se dilata
tags:
  - hilo
---
Toda la primera mitad de este Obsidian Notebook tiene el mismo agujero, y lo escribí yo mismo en [[HEX-01 El coste de construir un rival]] antes de tener nada construido.

*Me estoy montando un juez y el juez lo construyo con las mismas manos con las que construyo al acusado.*

Muy bien visto y completamente inútil, porque señalé el problema y luego me pasé otro mes midiéndolo todo contra versiones anteriores de mí mismo igualmente. Hay algo profundamente humano en diagnosticar un problema y no hacer absolutamente nada al respecto, y me consuela pensar que le pasa a bastante gente con cosas más importantes que un motor de Hex.

Esta nota es la que arregla eso.

## Lo que hace falta

Un rival que no comparta absolutamente nada con la red. Ni pesos, ni arquitectura, ni sesgos, ni puntos ciegos. Que no aprenda, para que no cambie entre entrenamientos. Y que sea determinista, para que dos mediciones separadas por un mes sigan siendo comparables.

O sea, un metro que no se dilata. De ahí el título, que tampoco me he roto la cabeza.

## De dónde sale

La condición de victoria en Hex es conectividad entre dos bordes, con lo cual la pregunta natural en cualquier posición es cuánto te falta para conectar.

Y eso resulta que se puede medir exactamente, cosa que a mí me pilló por sorpresa porque llevaba meses tratando la evaluación de posiciones como algo intrínsecamente difuso. Construyes un grafo sobre las `25` casillas donde

- tus propias fichas cuestan `0`, porque ya son tuyas y atravesarlas es gratis,
- las casillas vacías cuestan `1`, porque tendrías que gastar un turno en poner una ficha ahí,
- las fichas del rival cuestan infinito, porque por ahí no pasas ni con un pico.

El camino más barato de un borde tuyo al otro es entonces el número exacto de fichas que te faltan para ganar si el rival se quedara quieto mirando. Es literalmente un problema de camino mínimo, con lo cual Dijkstra te lo da y encima en microsegundos.

## El bot

En tu turno, pruebas cada casilla vacía, colocas tu ficha ahí, y calculas las dos distancias. Juegas la que maximiza

$$d_{\text{rival}} - d_{\text{mío}}$$

```python
board.board[row, col] = player
mine   = self._dijkstra(board, player)
theirs = self._dijkstra(board, rival)
board.board[row, col] = 0
score = theirs - mine
```

Fíjate en que las dos distancias se calculan **después de colocar tu propia ficha**, y ahí está el detalle del que depende que esto funcione. Si solo minimizas tu propio camino tienes un bot que corre en línea recta hacia su borde y se deja bloquear como un principiante entusiasmado. Al meter el término del rival, tu ficha se coloca donde además le estorba a él.

O sea que la defensa sale de la fórmula sin que nadie haya escrito una línea sobre defender. Me gusta bastante cuando pasa esto, y pasa muy poco. En `5x5` son como mucho veinticinco Dijkstras sobre veinticinco nodos, así que corre instantáneamente y no hace falta optimizar nada, cosa que agradezco y que está bien.

Y no es un rival flojo, que era lo que yo esperaba. La distancia mínima captura buena parte de lo que importa en un tablero pequeño. Lo que no captura son las amenazas dobles, esas donde tienes dos maneras de conectar y el rival solo puede cortar una, y ahí es donde un motor decente le saca ventaja. Exigente pero batible, que es exactamente lo que quieres de una vara de medir y lo contrario de lo que quieres de un examen.

## Dos deterministas producen una partida

Y aquí aparece el mismo problema de siempre con una cara nueva, porque este proyecto es fundamentalmente un chiste sobre determinismo que se repite cada dos notas.

El bot es determinista. Nash en inferencia también, porque elige por visitas máximas. Enfrentarlos produce **una** partida. Jugarla mil veces sigue produciendo esa misma partida. El resultado es un bit disfrazado de estadística, con su porcentaje y todo, muy convincente.

La solución no es meter azar, sino cambiar las posiciones de **salida**.

Fuerzas la primera jugada y las juegas todas. Veinticinco aperturas, cada una con los dos colores, cincuenta partidas distintas y todas deterministas y reproducibles hasta el fin de los tiempos. En el mundo de los motores de ajedrez a esto lo llaman una suite de aperturas y es como se ha medido siempre, o sea que llevaba décadas inventado y yo redescubriéndolo con cara de listo.

Lo bueno no es solo que tengas cincuenta partidas en vez de una. Es que el desglose por apertura te dice **desde dónde** pierde, cosa que un porcentaje agregado te esconde con muchísimo gusto.

Cuando lo corrí por primera vez, la red original perdía diez aperturas de veinticinco abriendo ella y **ninguna** respondiendo. Diez y cero. Esa asimetría estaba ahí desde el principio y no la habría visto nunca mirando un número resumen.

## Elo, otra vez, y esta vez bien

En [[HEX-12 Saber si esto ha servido de algo]] conté que encadenar comparaciones entre checkpoints adyacentes me daba `+1131` mientras la medición directa daba `+610`, y que el bueno era el segundo.

La forma correcta de hacerlo es no encadenar nada. Coges todas las partidas del torneo y ajustas todos los ratings a la vez, buscando los que maximizan la verosimilitud de haber observado esos resultados. Por dentro es una regresión logística donde el parámetro de cada jugador es su rating, y por fuera es un script de sesenta líneas.

```python
margin = SCALE * (ratings[white] + advantage - ratings[black])
loss = np.logaddexp(0.0, -margin) * won + np.logaddexp(0.0, margin) * (1.0 - won)
```

Tres detalles que importan y que están en [[LEM-elo]].

**Hace falta un ancla**, porque la verosimilitud solo depende de diferencias y sumarle mil a todo el mundo da exactamente el mismo valor. Anclo el bot de Dijkstra en cero, porque es lo único que no cambia entre entrenamientos.

**Hace falta un prior**, porque un jugador invicto se va a infinito muy alegremente si no hay nada que lo ate. Un gaussiano débil sobre los ratings basta y no distorsiona nada donde hay datos.

**Y conviene un parámetro para la ventaja de salida**, que en Hex es enorme. Si no lo pones, esa ventaja se reparte entre los ratings como buenamente puede y los contamina a todos por igual.

De propina, la inversa de la hessiana en el óptimo te da las barras de error, que es lo más guay.

## Los números

Setecientas cincuenta partidas, todas las aperturas forzadas con los dos colores entre cada par de jugadores, `200` simulaciones y la constante de exploración en `1.0` para todo el mundo, de modo que lo que se compara son las redes y no la ayuda que les doy en el examen.

| | contra el bot | Elo |
|---|---|---|
| v1, la original | `40 / 50` | `+207 ± 44` |
| v2, con buffer y simetría | `41 / 50` | `+318 ± 46` |
| v3 en la iteración `25` | `50 / 50` | `+606 ± 52` |
| v3 en la iteración `50` | `48 / 50` | `+692 ± 54` |
| v3 final | `49 / 50` | `+704 ± 55` |

![[elo_tournament.png]]

Vamos a ver qué dice esta gráfica.

**La distancia de v1 a v3 son casi quinientos puntos** y es real, con barras de cincuenta. Ahí sí me permito estar contento.

**De v2 a v3 hay `386` de esos.** El buffer y la simetría aportaron `111` y los tres arreglos de exploración aportaron el resto. Eso confirma que el diagnóstico de [[HEX-13 Doscientas veintiséis milésimas]] apuntaba al sitio correcto, y de paso confirma que si me hubiera quedado en v2 habría concluido que reentrenar no servía de nada y habría cerrado el proyecto con una conclusión falsa.

**Los tres checkpoints de v3 no son distinguibles entre sí.** `606`, `692` y `704` con barras de `52`. Todo lo que aprende, lo aprende en las primeras veinticinco iteraciones, y las setenta y cinco restantes no compran nada medible.

Eso último es el tipo de resultado que solo aparece si mides bien, y es el que más me ha hecho pensar. Llevaba todo el proyecto asumiendo que más iteraciones es mejor por definición, que es la misma lógica del que estudia catorce horas seguidas y cree que ha aprendido el doble que quien estudió siete.

## La ventaja de salida sale negativa

El ajuste devuelve `-46` para la ventaja del primer jugador.

Eso parece contradecir el teorema del robo de estrategia, que dice justamente lo contrario, y por un momento pensé que había encontrado algo. Spoiler número dos del cuaderno, no.

En estas partidas la primera jugada está **forzada y sorteada**, así que quien yo llamo primer jugador no elige su apertura, y el primer movimiento libre lo hace el otro. Lo que ese `-46` mide es que una apertura al azar vale un poco menos que la primera respuesta a ella.

Es un artefacto del diseño y a la vez es informativo, y lo dejo escrito aquí para que nadie piense que he refutado a Nash desde un portátil en Cádiz. Ojalá.

## El metro también se acaba

Contra el bot, v3 gana `49` o `50` de `50` según el checkpoint. O sea que está saturado y ya no distingue entre las versiones nuevas.

Sirve perfectamente para contar la historia de v1 a v3 y no va a servir para medir la siguiente mejora. Cuando construyes una vara de medir con la que tu sistema se sale del rango has llegado a un sitio bueno, y a la vez te has quedado sin instrumento, que es una forma bastante elegante de tener un problema nuevo.

Lo que tocaría es un solucionador exacto. Hex en `5x5` es lo bastante pequeño para resolverlo con alfa-beta y memoización sobre el mismo union-find que ya tengo, y eso permitiría medir contra el juego perfecto en vez de contra otra cosa que también escribí yo.

Y ahí sí desaparecería el agujero que señalé en la primera nota. Cuatro meses y pico después, pero desaparecería.

Y por último, [[HEX-16 Liquidación por cierre]].
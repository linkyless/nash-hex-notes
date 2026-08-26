---
title: HEX-16 Liquidación por cierre
tags:
  - hilo
---
El momento en el que esto funcionó fue una línea de texto en una terminal a las cinco y veinte de la mañana que decía `49/50`, y yo mirándola como se mira una notificación del banco.

Ni había música ni había nadie delante. El portátil hacía el mismo ruido de ventilador que lleva haciendo desde junio y probablemente seguirá haciendo hasta que se muera.

Esperaba algo más. No sé exactamente qué. Supongo que esperaba que estos meses de código, gráficas, errores de signo, simulaciones y tal sirvieran de algo. O yo que sé, me animaran a luchar contra la vida. No lo hicieron.

Así que, en vez de escribir una nota sobre lo bonito que ha sido todo, que sería verdad a medias y además bastante aburrido, voy a hacer lo que se llama "inventario".

## Lo que no es mío

La web la diseñé yo. Tenía bastante claro cómo quería que se viera: las texturas, la paleta de colores, el tablero, el estilo general.

Lo que no me apetecía era pasar dos días averiguando por qué un hexágono tiene un recorte distinto en la fila de abajo y por qué el canvas decide que una línea está medio píxel a la izquierda. Tampoco me apetecía montar desde cero toda la comunicación navegador-servidor para una aplicación que, sinceramente, no era la parte del proyecto que quería aprender.

Eso se lo dejé a Claude Code y seguí con lo demás. Lo que sí es mío de la web cabe, más o menos, en tres decisiones.

**El servidor no guarda estado.** El navegador conserva la lista de movimientos y la manda completa en cada petición, y el servidor reconstruye la partida desde cero antes de responder. En un tablero `5x5` eso cuesta prácticamente nada y, a cambio, deshacer, compartir una posición o reiniciar salen gratis.

**El overlay también es mío**. Poder ver dónde ha gastado la búsqueda sus simulaciones y, por otro lado, ver la policy cruda de la red sin búsqueda acabó siendo bastante más útil de lo que esperaba. Lo hice porque quedaba bonito. Luego fue prácticamente un instrumento de depuración.

**La barra de evaluación tenía además que estar siempre desde el punto de vista del humano**. El value sale desde la perspectiva de quien mueve en ese momento, así que cuando le toca a Nash hay que invertirlo. Si no, la barra cambia de lado cada turno y parece que el motor está jugando consigo mismo, que técnicamente es lo que está haciendo, pero no es lo que quiero enseñar.

**Los gráficos de matplotlib siguen el mismo principio**. La decisión de qué medir y qué significa cada curva es mía. La línea de referencia en `ln 25` que aparece en la gráfica de pérdidas es de hecho una de las mejores decisiones del proyecto. El código que hace que matplotlib dibuje hexágonos perfectamente alineados, respete la paleta y no convierta cada figura en una discusión personal lo hizo Claudia.

**Los tests funcionan igual**. Yo decidí qué tenía sentido comprobar. El andamiaje de pytest lo pedí. Y, por una ironía bastante fea, el test que había escrito para comprobar el signo del value estaba mal planteado. El episodio entero está en [[HEX-11 Cuanto mejor, peor]].

Todo lo demás es mío.

## El inventario de lo que sigue roto

Y aquí toca sacar las cajas a la acera.

No hay gating. Cada checkpoint se guarda sin comprobar nada, así que una iteración mala puede pasar directamente a la siguiente. Ahora tengo una medición que demuestra que esto importa: los tres checkpoints finales de v3 son prácticamente indistinguibles y la mejora medible ocurre muy pronto, alrededor de las primeras `25` iteraciones.

El value se satura. Termina alrededor de `0.949`, y como las etiquetas son exactamente `-1` y `+1` y Hex no tiene empates, no existe ningún incentivo interno para que esa salida vuelva hacia el centro.

El bot de Shortest Path de Dijkstra también está agotado. Gana `49` o `50` de `50` contra cualquier checkpoint reciente, lo cual significa que construí una regla para medir el sistema y el sistema terminó saliéndose del rango de la regla.

La siguiente cosa sensata sería un solver exacto. `5x5` es lo bastante pequeño como para plantear alpha-beta con memoización sobre el mismo union-find que ya tengo y, de esa forma, medir contra juego perfecto en lugar de medir contra otro programa que escribí yo.

Esa es probablemente la debilidad grande del proyecto entero y ya la había señalado en el [[HEX-01 El coste de construir un rival]] sin tener ni idea de cuánto me iba a costar.

Cada número que he sacado hasta ahora es relativo a algo que también construí yo.

El baseline externo reduce el problema bastante. Lo que no hace es eliminarlo.

Después están las cosas más aburridas: `50` partidas por emparejamiento dejan barras de error de alrededor de `±50` Elo, así que una buena parte de las diferencias que parecen distintas no lo son de forma estadísticamente convincente. Decir eso en voz alta es bastante menos espectacular que enseñar una cifra de Elo enorme, pero también es la diferencia entre enseñar un número y enseñar algo que el número realmente sostiene.

Queda la evaluación por lotes, porque ahora mismo consulto la red de una posición cada vez mientras la GPU está ahí mirando la situación con bastante indiferencia y el cuello de botella sigue siendo un árbol en Python interpretado.

Y quedan los bloques residuales. En `5x5` no me hacen ninguna falta. En `11x11`, probablemente sí.

Sale una lista bastante larga, pero sale larga porque ahora sé mirar, no porque el proyecto esté peor.

## Las cosas que aprendí y que no van en el README

En cuatro meses, nada falló nunca de forma visible. Ni una excepción sin cazar, ni un test en rojo, ni un _stack trace_ manchando la terminal.

Yo pensaba que el muro iba a ser la matemática o el código y creo que lo difícil resultó ser que nada te gritara al oído para avisarte.

La métrica que miras es la métrica que optimizas, aunque no quieras y te creas por encima de eso. Yo abrazaba mi 97% de winrate como un trofeo, mientras el motor perdía consistentemente contra una fila rellenada de izquierda a derecha por un humano que llevaba diez minutos jugando. Los dos datos eran matemáticamente ciertos. El problema es que yo solo quería mirar el primero, y eso está mal.

Un test en verde apaga una zona del código en tu cerebro. No la vuelves poco probable, directamente la borras de tu mapa.

Razonar sobre la topología abstracta de una fórmula no es lo mismo que meterle tus putas constantes. Todo el mundo te dice en los papers que he visto que el prior se disuelve asintóticamente como $\mathcal{O}(1/N)$ y que la búsqueda bruta acaba imponiéndose. Y es verdad. El pequeño detalle es que, con mis constantes específicas, hacían falta cuatro mil simulaciones por turno para que eso pasara en el mundo real. La asintótica es cierta, pero a ti te tiene que importar el régimen finito en el que vives.

Y por último, mejorar un componente aislado puede empeorar el sistema entero. Como tenía un signo invertido, cuanto mejor y más precisa se volvía mi _value head_ evaluando posiciones, con más convicción el MCTS decidía inmolarse eligiendo el lado equivocado. La red aprendía la geometría del juego a la perfección, y usaba ese conocimiento exactamente al revés. La versión de la red sin entrenar ganaba sus partidas porque el ruido estadístico puro no tiene una dirección concreta hacia el suicidio.


## Lo único que demuestra la tesis

Y ahora, para ir echando el cierre, la única parte de todo este experimento que no es un simple porcentaje flotando en memoria.

En algún punto oscuro del entrenamiento, si miras el mapa de calor de las aperturas iteración tras iteración, ves cómo la distribución de probabilidad converge de forma implacable hacia el centro exacto del tablero.

En Hex, el centro es la casilla más fuerte de la geometría. Está a la misma distancia de los cuatro bordes y participa en más conexiones potenciales cruzadas que ninguna otra baldosa. En torneos serios humanos, abrir en el centro se considera una ventaja tan brutal que es la jugada estándar que tu oponente te va a robar invocando la regla del pastel.

Nadie le explicó esto a Nash. No hay ni una triste instrucción condicional hablándole del centro. No le enchufé un dataset con partidas de grandes maestros. No hay libros de aperturas. Ese conocimiento emergió de la puta nada. Salió de jugar mil quinientas partidas contra sí misma en el vacío y de deducir, solo a base de retropropagar gradientes, que las posiciones donde controlaba esa baldosa acababan ganando. Eso es lo que yo buscaba cuando empecé todo esto y que dejé escrito literalmente en la primera nota de Obsidian. Un sistema donde nadie supiera la respuesta correcta —ni yo, ni un conjunto de datos, ni ningún humano que haya pisado la Tierra— y que aun así, iteración tras iteración, mejorase.

## Y ya

Tengo un repositorio, una web desplegada donde entra poca gente a perder, y un cuaderno con dieciséis notas interconectadas donde he documentado de forma casi masoquista todas las veces que me equivoqué de camino.

Ese último punto es curiosamente el que más me ha sorprendido de todo el asunto. Salió mucho mejor la parte de contarlo que la parte de programarlo, y no lo digo con amargura. Lo digo porque el código en sí funciona en un tablero asfixiante de 5x5, pero saber explicar y entender por qué todo se fue a la mierda es algo que me puedo llevar puesto a cualquier sitio.

He acabado entendiendo más cosas mirando cómo fallaba el motor que mirándolo ganar.

Y probablemente eso sea lo único que me llevo de aquí que no depende de que el tablero tenga cinco por cinco casillas.
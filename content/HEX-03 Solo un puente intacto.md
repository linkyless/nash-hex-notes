---
title: HEX-03 Solo un puente intacto
tags:
  - hilo
---
# Las reglas del Hex

Si bien Hex me recuerda al nombre de aquel videojuego de Daniel Mullins, en este caso es un juego de mesa que se juega entre dos personas, con piedras si se puede y si no, pues con lo que haya.

El tablero es un rombo de casillas hexagonales. Dos jugadores alternan turnos colocando una ficha en cualquier casilla vacía. Las fichas no se mueven, no se capturan y no se promocionan. Uno de los jugadores gana si conecta el borde norte con el sur mediante una cadena de fichas suyas, y el otro gana si conecta el este con el oeste.

Ya está: ese es el juego entero. No es broma, ojalá lo fuera, porque a estas alturas del cuaderno me vendría bien tener más cosas que explicar.

![[nash-initial-pos.png]]

Lo inventó Piet Hein en 1942 y lo redescubrió el maravilloso John Nash por su cuenta unos años después, cosa que en matemáticas históricamente suele pasar. Y digo matemáticas porque la mayoría de los juegos tienen un componente matemático bastante fuerte, y porque toda esta peña era así de guay. Ya conocemos muchas historias. Se vendió comercialmente con el nombre de Con-tac-tix, que suena a marca de tiritas y probablemente explique por qué no lo conoce nadie. Encima se parece a Tic-Tac-Toe, ¡plagio! ¡plagio!, aunque no sé quién salió primero.

## Lo de los empates

En Hex es imposible que una partida acabe en tablas. Ya lo dije en [[HEX-01 El coste de construir un rival]], pero conviene entender por qué, porque el motivo tiene consecuencias directas sobre el código.

Cuando el tablero se llena, alguien ha conectado sus dos bordes. No hay ninguna configuración de fichas donde nadie gane. Y tampoco hay ninguna donde ganen los dos, porque una cadena norte-sur (arriba-abajo) y una cadena este-oeste (derecha-izquierda) tendrían que cruzarse en algún punto y solo cabe una ficha por casilla.

Para el motor esto es un regalo. La función que evalúa una posición terminal no necesita rama de empate, y el resultado de cualquier partida es siempre `+1` para alguien y `-1` para el otro. Cuando en [[HEX-09 Self-play]] toque etiquetar posiciones con el resultado, no habrá que decidir qué hacer con las tablas, sencillamente porque no existen.

Piénsalo desde el otro lado un momento. En ajedrez, la existencia de las tablas te obliga a tener una tercera etiqueta, y esa tercera etiqueta se lleva por delante una parte enorme de las partidas de alto nivel, y de repente tu red tiene que aprender a predecir un empate, que es un concepto mucho más sutil que ganar o perder. Aquí eso no pasa. El espacio de resultados tiene dos elementos y ninguno de los dos es ambiguo.

## Una cosita sobre ser el primer jugador

Con juego perfecto, el que abre gana siempre. Y esto no viene de mirar muchas partidas, es un teorema con una demostración de tres líneas que está en [[LEM-robo de estrategia]]. En realidad, se podría demostrar a través de estados de W y L en un DAG pero no nos metemos ahí porque no somos tan estúpidos.

Recomiendo leerla aunque sea por el gusto de ver un argumento que no construye absolutamente nada y aun así demuestra que algo existe. Es como, "venga, imagínatelo por lo menos, porque sé que tengo razón". Y es que al final la tiene.

Esto puede parecer guay, lo de ganar siempre siendo el primer jugador si juegas perfecto, pero la consecuencia práctica es fea. Nash (el motor) va a parecer muchísimo más fuerte cuando abra él que cuando abra el rival, y eso no dice nada sobre lo bueno que es. Toca acordarse de esto cuando llegue el momento de medir.

## El puente

Coge dos fichas tuyas separadas por dos casillas vacías, colocadas de forma que ambas casillas vacías sean vecinas de las dos fichas. Esas dos fichas están conectadas. Aunque no lo parezca, y aunque haya un hueco visible entre ellas. Esto es porque si el rival ocupa una de las dos casillas, tú ocupas la otra y la cadena se cierra igual. El rival no puede romper la conexión, solo puede elegir por dónde la completas tú, que es una forma bastante humillante de participar en una partida.

![[neighborsoff.jpg]]

Un jugador decente de Hex ve grupos que ya están conectados a través de puentes que todavía no ha jugado, y buena parte de la estructura de la partida es virtual hasta que alguien la ataca. Es un juego donde la mitad de lo que existe está en potencia, como diría Aristóteles.

Si juegas Rocket League y llevas un tiempo en ello, dejas de ver un coche y una pelota y empiezas a ver rotaciones que todavía no han ocurrido, y hasta puedes soñar con ello. ¿Cómo es posible este nivel de Game Sense, nen?

Lo importante es que nadie le va a explicar al motor lo que es un puente. No hay ninguna línea de código que diga que esa forma es buena. Si el sistema funciona, la red tendrá que descubrirlo sola de tanto ver que las posiciones con esa geometría acaban ganando.

Cuando en [[HEX-07 Dos cabezas]] hable de lo que detectan los filtros, este es el tipo de cosa que espero encontrar. Aunque sospecho que si aparece, va a aparecer de una forma que no reconozca al mirarla. Por supuesto, quizás la red esté haciendo algo bastante más raro y "puente" sea una palabra mía que a ella no le sirva para nada.

## Cinco por cinco

Todo el proyecto va sobre `5x5`. Es pequeño, es casi un juguete, y un humano decente lo lee entero sin despeinarse.

Estuve tentado de irme a `9x9` para que la cosa sonara más seria, y la tentación duró justo lo que tardé en hacer la cuenta. Cada movimiento son unos cientos de simulaciones, cada partida unos veinte movimientos, cada iteración cincuenta partidas, y todo eso multiplicado por treinta iteraciones. En `5x5` sale un número con siete cifras y dos horas de reloj. En `9x9` sale otra cosa, y encima serían días esperando a ver si el código tenía un bug.

Prefiero un sistema pequeño que funcione y del que pueda demostrar que funciona, antes que uno grande a medio entrenar del que solo pueda decir que va prometiendo.

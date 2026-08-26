---
title: LEM-robo de estrategia
tags:
  - lema
---
En Hex, con juego perfecto, gana el primer jugador. Siempre.

La demostración es de John Nash, ocupa tres líneas, y no construye absolutamente nada.

## El argumento

Supóngase que el **segundo** jugador tiene una estrategia ganadora. Llámese $S$.

El primer jugador coloca una ficha en cualquier sitio y a partir de ahí juega $S$ como si fuera el segundo, ignorando esa primera ficha. Cuando $S$ le pida una casilla que ya tiene ocupada por ella, coloca otra en cualquier sitio libre y sigue.

Resultado, el primer jugador está jugando la estrategia ganadora del segundo con **una ficha extra** en el tablero. Y en Hex una ficha de más nunca estorba, porque solo puede ayudar a conectar y jamás romper una conexión propia.

Luego el primero gana. Pero habíamos supuesto que el segundo tenía la estrategia ganadora, y solo uno de los dos puede tenerla. Contradicción.

## Dónde se apoya

**En que no hay empates**, que es lo que garantiza que exactamente uno de los dos tenga estrategia ganadora, y que es un teorema aparte equivalente al del punto fijo de Brouwer.

**En que una ficha de más nunca perjudica**, cosa específica de Hex. En ajedrez un movimiento de más puede arruinarte y en Go una piedra mal puesta puede matarte un grupo.

Quita cualquiera de las dos y el argumento se cae entero.

## Por qué no sirve de nada

Demuestra que la estrategia existe y no da ni una pista de cuál es. Es una demostración no constructiva, con lo cual no se puede escribir en código.

Lo cual, pensándolo, es lo que hace que este proyecto tenga sentido. Si el robo de estrategia diera la estrategia, no habría nada que aprender.

## Lo que sí cambia

Cualquier medición donde un motor abra siempre está midiendo la ventaja de abrir y no la calidad del motor. Se corrige alternando colores sobre las mismas aperturas.

Y la **regla del pastel**, que es cómo lo arreglan los humanos. El primero coloca una ficha y el segundo puede robársela, quedándose con esa jugada y cambiando de color. Así que al primero le conviene abrir con algo mediocre, porque si abre demasiado bien se lo quitan. Es repartir la tarta y dejar que el otro elija trozo.

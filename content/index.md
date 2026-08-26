---
title: Nash, un motor de Hex que aprende por su cuenta
tags:
  - indice
---
Demo: https://nash-hex-cf6e7dc1bd23.herokuapp.com/

Se busca construir un motor que juegue a Hex sin que nadie le enseñe a jugar, y hacerlo entendiendo cada pieza en vez de copiar una implementación de las que ya hay, ni tampoco usando IA para ello, que me apetece aprender de verdad, no entregar trabajos. El resultado, cuando llegue, será algo que juega mejor que yo a un juego que yo tampoco domino porque no lo había visto nunca ni conozco un libro que me diga cómo jugar perfectamente, lo cual dice cosas bastante incómodas sobre dónde está el listón.

Aquí escribo antes, durante y después del proyecto. El orden no importa demasiado, porque lo que importan son las notas. Terminé y regresé muchas veces. No he reordenado el camino para que quede más limpio de lo que fue, entre otras cosas porque el camino tuvo dos errores de signo que me tuvieron dos días mirando el sitio equivocado, y esa parte me parece de las mejores del proyecto. Y luego hay una más adelante que no era un bug y que me destruía todo igualmente, así que está bien.

## Hilo

[[HEX-01 El coste de construir un rival]]
[[HEX-02 Memoria episódica sin estados intermedios]]
[[HEX-03 Solo un puente intacto]]
[[HEX-04 Cuatro nodos que no existen]]
[[HEX-05 La arquitectura de la cosa]]
[[HEX-06 MCTS]]
[[HEX-07 Dos cabezas]]
[[HEX-08 Cuánta intuición hace falta para ser experto]]
[[HEX-09 Self-play]]
[[HEX-10 Algo anda mal]]
[[HEX-11 Cuanto mejor, peor]]
[[HEX-12 Saber si esto ha servido de algo]]
[[HEX-13 Doscientas veintiséis milésimas]]
[[HEX-14 Tres arreglos que no son intercambiables]]
[[HEX-15 Un metro que no se dilata]]
[[HEX-16 Liquidación por cierre]]

## Lemas

Estas son cosas que hacen falta para entender el hilo y que estorban si las meto dentro. Un lema es ese resultado auxiliar que tienes que demostrar antes de poder demostrar lo que de verdad querías, y esto es justamente eso. Van escritos en un registro bastante más seco que el hilo, y es a propósito.

[[LEM-bandido-multibrazo]]
[[LEM-union-find]]
[[LEM-el-grafo-computacional]]
[[LEM-filtros-y-canales]]
[[LEM-campo-receptivo]]
[[LEM-no-linealidad]]
[[LEM-softmax-enmascarado]]
[[LEM-tanh]]
[[LEM-entropía]]
[[LEM-forma-canónica]]
[[LEM-robo-de-estrategia]]
[[LEM-elo]]
[[LEM-adam]]
[[LEM-temperatura]]
[[LEM-ruido-de-dirichlet]]
[[LEM-replay-buffer]]
[[LEM-camino-minimo-en-hex]]
[[LEM-deepcopy-y-perfilado]]

## ¿Qué logramos con esto?

Un tablero de `5x5`, cien iteraciones de partidas contra sí mismo, y un motor que gana `49` de las `50` aperturas forzadas contra un oponente de camino mínimo que no comparte ni un solo parámetro con la red. Poco más de una hora en un portátil, y la tarjeta gráfica no se entera de nada porque ni se toca, cosa que también tiene su explicación.

En Elo, ancladas contra ese oponente, las tres versiones del motor quedan en `+207`, `+318` y `+704`. La distancia entre la primera y la última son casi quinientos puntos, y de esos, `386` los aporta arreglar un solo problema, que es que la exploración de la búsqueda llevaba noventa y nueve iteraciones apagada sin que nada lo indicara.

Hubo un `97%` en algún momento anterior que era más alto que todo esto y a la vez peor, cosa que suena a excusa y está medida. Está en [[HEX-12 Saber si esto ha servido de algo]] y es la parte del proyecto que más me costó aceptar.

Y en algún punto del entrenamiento descubrió por su cuenta que el centro del tablero es la apertura más fuerte de Hex, que es correcto y que nadie le dijo. De todo lo que ha salido de aquí, eso es lo único que de verdad demuestra que la idea funciona.

Hay una web donde se puede jugar contra él. Es la parte del proyecto que menos me interesaba y por tanto la que menos escribí, y eso está contado tal cual en [[HEX-16 Liquidación por cierre]] sin fingir lo contrario.

![[finishedgame.png]]

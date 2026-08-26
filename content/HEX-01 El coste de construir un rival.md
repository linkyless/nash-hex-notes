---
title: HEX-01 El coste de construir un rival
tags:
  - hilo
---
![[2- Programación/Hexodus/hex1.png]]

El proyecto anterior fue una red neuronal desde cero, en numpy, que reconoce dígitos escritos a mano. Salió bien. Acierta un `97%` de las veces y hay un visualizador donde dibujas con el ratón y ves las diez barritas moviéndose en tiempo real, que es exactamente el tipo de cosa que funciona muy bien en un clip de TikTok de quince segundos y bastante peor cuando alguien te pregunta qué hace por dentro.

Y me dejó con una sensación rara que tardé en poder nombrar. La red hace aquello para lo que la entrené y ni un milímetro más. Le di sesenta mil imágenes con la respuesta escrita al lado y aprendió a copiar la respuesta, que es literalmente lo que le pedí. El techo lo puso la persona que etiquetó ese dataset hace treinta años, a mano, y mi red no va a reconocer un siete mejor de lo que lo reconocía ella.

Lo que quiero ahora es lo contrario. Un sistema donde nadie sepa la respuesta correcta, ni yo, ni un dataset, ni ningún humano que haya existido, y que aun así mejore. Eso, en particular, está inventado desde 2017 y se llama **AlphaZero**. Lo que se hizo famoso fue que ganara al Go, cosa que a mí me da bastante igual porque en este caso no quiero recrearlo. Lo que me interesa es el mecanismo, y hay suficientes papers que lo explican, pero es bastante simple: el programa arranca con pesos aleatorios, juega contra sí mismo, y de ahí sale absolutamente todo lo demás. No hay ni una partida humana, ni un libro de aperturas que te indiquen cuál es la forma más correcta de empezar a jugar, ni una función de evaluación escrita por un grandmaster, lo único que tendría el programa como mucho a su disposición son las reglas del juego.

## Elegir el juego

Lo primero que se me ocurrió fue ajedrez, que habría sido una idea preciosa durante unas seis horas hasta que me acordara de que existen el enroque, la coronación, la captura al paso y una regla de repetición de tablas que —fíjate que se me da bien jugar al ajedrez—, siempre se me olvida cuántos movimientos iguales se han de hacer para empatar. Cada una de esas reglas es una superficie más donde meter la pata, y yo necesitaba meter la pata en otros sitios, aunque me sirviera como proyecto divertidísimo para este verano de primero de carrera.

Descartado eso, empecé a mirar juegos pequeños, y ahí apareció Hex.

![[hexboard.png]]

La primera propiedad que me convenció fue el tamaño. En un tablero de `5x5` hay veinticinco casillas, lo cual ya genera un espacio de estados que no cabe en ningún ordenador, y sin embargo es lo bastante manejable para que unos cientos de simulaciones digan algo útil. En `11x11`, que es el tamaño con el que se juega de verdad, esto mismo tardaría días.

La segunda me llevó un rato más. Resulta que en Hex es imposible que una partida acabe en tablas. Cuando lo leí me pareció razonable si lo pensaba un poco, porque el tablero se va llenando y algo tiene que pasar. Luego lo pensé un poco más y me pareció una errata, porque no veía por qué no podía quedar todo bloqueado como en tantos otros juegos. Y no es una errata. Es un teorema, y además resulta ser equivalente al teorema del punto fijo de Brouwer, que es de esas conexiones entre un juego de mesa y la topología que uno agradece haberse encontrado.

La tercera es puro egoísmo. La condición de victoria en Hex es una consulta de conectividad, y llevo tres años haciendo consultas de conectividad en programación competitiva. Empezar un proyecto donde la primera pieza difícil resulta ser la única que ya sé construir tiene un valor psicológico que no pienso despreciar.

Y la cuarta la vi bastante después, cuando ya estaba metido. Las reglas de Hex caben en una frase, y eso significa que el motor las implementa en una tarde y el resto del tiempo lo dedicas al problema de verdad.

O sea que habíamos dicho que necesitaba un juego pequeño, sin empates, con victoria por conectividad y con reglas cortas. Escrito así parece que hice una lista y busqué el juego que encajara, cuando en realidad encontré Hex y luego fui descubriendo que cumplía cosas. Pensé también en el Connect 4, pero bueno, al final te decantas por una cosa y no por otra.

## Lo de medir

Si construyes un sistema que aprende solo, no tienes contra qué medirlo.

En MNIST hay un conjunto de test con las respuestas al lado. Esas respuestas suelen ser o "ES CORRECTO" o "NO ES CORRECTO": una decisión binaria. Entonces el proceso es que básicamente se saca un número, lo comparas con el de otra gente que también ha usado MNIST, y ya está. Pero claro, aquí no hay respuestas, así que para saber si mi motor mejora la única referencia disponible soy yo mismo en una versión anterior. Que también la hice yo, y que, por supuesto, también puede estar mal.

Me estoy montando un juez y el juez lo construyo con las mismas manos con las que construyo al acusado. Funciona, con matices, y esos matices se llevan por delante casi dos semanas cuando salen mal. Está en [[HEX-12 Saber si esto ha servido de algo]] y en las dos notas de errores.

## Duda existencial

Si el sistema tiene que descubrirlo todo solo, ¿cuánto puedo ayudarle antes de que deje de contar? Si lo pienso, claro, yo elijo la arquitectura de la red, elijo cuántas simulaciones hace, elijo cómo se representa el tablero. Pero, ¿no sigue siendo eso una decisión que tomo yo? Escribir "los puentes son buenos" en el código sería hacer trampa con todas las letras, por supuesto. En cambio, elegir tres capas convolucionales en lugar de dos claramente no lo es. Y entre esas dos cosas hay una zona gris bastante ancha donde meto la forma canónica, que es un truco que le ahorra a la red la mitad del trabajo de aprender y que le he dado yo, gratis, porque me apetecía.

No tengo respuesta y sigo adelante igualmente, que suele pasar bastante.

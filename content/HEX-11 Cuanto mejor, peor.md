---
title: HEX-11 Cuanto mejor, peor
tags:
  - hilo
---
![[wrong.jpg]]
## El montaje

Con el bug de la trasposición arreglado seguía perdiendo contra una red virgen. `43%`.

Sabía dos cosas que antes no sabía. Que el problema está ahí desde la primera iteración, y que el value head funciona bien mientras la policy no.
Y sabía una tercera que era falsa, aunque en ese momento la habría defendido con vehemencia, y es que el signo del value estaba comprobado.
Porque yo había escrito un test para eso. Como haría cualquier persona responsable que ha leído tres artículos sobre buenas prácticas y ahora está convencida de que sabe hacer testing.

Así que hice testing diciéndole a Claude que me hiciera el testing exacto que necesitaba yo.

## El test

Montaba una posición donde el jugador que movía tenía una victoria en un movimiento. Corría la búsqueda. Imprimía el `mean_value` de cada hijo de la raíz.

El razonamiento es limpio. Si el signo estuviera invertido, la jugada que gana la partida tendría el valor más **bajo** de todos, porque el sistema la estaría leyendo como una catástrofe. Con lo cual mirando el orden ya sabes si el convenio está bien.

Ejecuto. La jugada ganadora sale con `mean_value` de `1.0` y se lleva `191` de `200` visitas.

`1.0`. `191/200` visitas. Verde. Continué con mi vida (estaba mal).

## Lo que ese verde me hizo

Un día entero (en realidad una madrugada entera) mirando en otras direcciones.

Miré la constante de exploración. Miré el tamaño del buffer. Miré si la policy había colapsado, otra vez, por si acaso. Miré la inicialización de los pesos. Miré cosas que no tenían absolutamente nada que ver con nada como, por ejemplo, cómo programar con buen Clean Code en la era de la IA, porque estoy en la mierda y mi código seguramente tenga algo que no veo porque no sé programar bien.

En **ningún momento** volví a mirar el *backup*, porque el backup estaba comprobado. Esto es un efecto psicológico bastante fuerte que yo no había notado nunca antes. 

## Por qué mintió

La respuesta está en una línea de cuatro caracteres que escribí sin pensarla dos veces, meses antes, y que además es correcta.

Cuando la búsqueda llega a un nodo terminal, `_expansion` no consulta la red. Devuelve `1.0` a pelo, porque en Hex quien movió para llegar a un nodo terminal es siempre el que ganó, y con mi convención de perspectiva eso es `+1` sin condicionales.

Ese `1.0` está en la convención correcta.

Y mi test montaba una posición con **victoria en un movimiento**, o sea que la jugada que estaba comprobando era exactamente una que llega a un nodo terminal.

Con lo cual mi test verificaba el único caso del sistema que no pasa por la red. Las victorias inmediatas se propagaban perfectamente. Todo lo demás iba invertido.

Escribí un detector de incendios y lo instalé en la única habitación de la casa que era ignífuga. Y luego, durante un día entero, cada vez que olía a humo me decía que no podía ser, porque el detector estaba en verde.

## El error, ahora sí

Traza el value desde que sale de la red hasta que se guarda, despacio, que es lo que tendría que haber hecho el primer día.

`_expansion` recibe un nodo. Le pasa a la red el tablero **canónico** de ese nodo, donde el `1` es quien mueve **ahí**. Así que el value que devuelve está desde la perspectiva de quien mueve en ese nodo.

Ese value se lo pasa al backup, que lo suma tal cual al `value_sum` de ese nodo.

Pero mi convención dice que el `value_sum` de un nodo se mide desde la perspectiva de **quien movió para llegar** a él.

Y esa persona es el rival de quien mueve ahora.

Personas opuestas. Signo invertido. Cada evaluación de la red entrando al árbol con el sentido cambiado, todas, durante dos horas de entrenamiento.

## La consecuencia

Aquí está lo que hace que este error merezca una nota entera en vez de un párrafo, y es que produce un comportamiento que rompe la intuición más básica que uno tiene sobre por qué merece la pena arreglar cosas.

Cuanto mejor se volvía el value head juzgando posiciones, **más consistentemente elegía la búsqueda el lado equivocado de ellas**.

La red virgen ganaba porque su value era ruido, y el ruido no tiene dirección. Un ruido invertido sigue siendo ruido, te da exactamente igual multiplicarlo por menos uno.

Pero conforme el entrenamiento hacía que el value evaluara mejor de verdad, y lo estaba haciendo, y las gráficas lo confirmaban, el sistema se equivocaba con cada vez más convicción.

O sea que el entrenamiento funcionaba perfectamente. La red aprendía perfectamente. Lo que pasaba es que el árbol usaba lo aprendido para hacer justo lo contrario, y por tanto **cuanto más aprendía, peor jugaba**. El modelo estaba mejorando, y el sistema estaba empeorando, y ambos tenían razón.

Es la primera vez en mi vida que me encuentro con un sistema donde mejorar un componente empeora el conjunto. Sigo pensando que es de las cosas más raras que he visto programando.

## El arreglo

Negar el value al salir de `_expansion` y reescribir el backup con alternancia limpia, subiendo por los punteros a padre y multiplicando por `-1` en cada nivel. Después de eso, el mismo entrenamiento, los mismos parámetros, la misma red, y `97` victorias de `100` contra la iteración cero.

## El dilema de las dos convenciones

Dos errores de signo, ninguno de los cuales rompió nada, ambos escondidos en la única parte del sistema que ningún test podía tocar.

Y aquí hay algo que me da vueltas. Ninguno de los dos habría existido si en [[HEX-06 MCTS]] hubiera elegido la otra convención de perspectiva, decisión que tomé en dos minutos porque me pareció que daba exactamente igual cuál.

Existe una versión de este proyecto donde elijo la otra convención, los dos bugs no aparecen, el entrenamiento sale bien a la primera y yo escribo un README triunfal en septiembre. En esa versión habría una gráfica bonita, tres capturas y probablemente una confianza bastante injustificada. En esa versión el motor probablemente juega parecido a este. Y yo he aprendido bastante menos. Así que no tengo del todo claro cuál de las dos versiones preferiría, aunque sí tengo claro cuál de las dos me habría gustado vivir.

Y con `97` de `100` yo di el proyecto por terminado. Escribí en algún sitio que solo quedaba el README. Fue una afirmación bastante optimista porque, naturalmente, NO quedaba solo el README. 
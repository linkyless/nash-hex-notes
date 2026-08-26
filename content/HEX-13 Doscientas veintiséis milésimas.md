---
title: HEX-13 Doscientas veintiséis milésimas
tags:
  - hilo
---
Un día después de pensar que todo estaba terminado, Raúl (un game tester bebé pequeño de 19 años ahora mismo a día 24 de agosto de 2026 amigo y padre y hermano mío) estaba jugando contra Nash en la web y encontró una línea sutil que lo mataba. No la de [[HEX-12 Saber si esto ha servido de algo]], una nueva, con una defensa que el motor sencillamente no veía.

Lo primero que hice fue lo de siempre, que es subir las simulaciones a ver qué pasa. La solución del que no tiene ni idea de qué está pasando y confía en que la fuerza bruta le tape el agujero.

Con `500` no bloqueaba. Con `1000` tampoco. Con `2125` seguía sin bloquear. Con `2275` bloqueaba.

Fui estrechando el intervalo por bisección y haciendo búsquedas binarias, y el corte estaba en `2225`. Una simulación por debajo, muerte. Una por encima, defensa perfecta.

Y ahí me acordé del `0.33` contra `0.34` de la nota anterior y pensé que dos escalones no son casualidad, son un mecanismo que no he entendido y que lleva bastante tiempo riéndose de mí. Esta nota es el mecanismo. También es la nota donde tengo que desdecirme de tres cosas que escribí antes, así que empecemos por ahí, que es lo más incómodo y me lo quito de encima.

## Tres cosas que escribí y que son falsas

**Falsa #1.** Escribí que la mezcla uniforme impedía que el MCTS explorara jugadas de prior bajo durante el entrenamiento, y que por eso esas posiciones nunca entraban en el dataset. Mentira. Está exactamente del revés. La mezcla uniforme es lo **único** que estaba explorando esas jugadas. Sin ella, el prior de la casilla del bloqueo es del orden de una diezmilésima y no la visitaría jamás, con ningún presupuesto que quepa en una tarde ni en una vida. Estaba culpando al parche del problema que el parche era lo único que contenía. Bravo. Magnífico. Otro gran momento para la ingeniería de software.

**Falsa #2.** Escribí que la transición era probabilística. Y una mierda. Si en inferencia no metes ruido y eliges por visitas máximas, el MCTS es completamente determinista, con lo cual `2225` no es un valor esperado, es un umbral bastante fuerte. Corre la posición tres veces y te da lo mismo las tres. Yo lo llamé transición probabilística porque suena mejor y porque en ese momento no había comprobado nada, que son dos motivos que suelen ir juntos.

**Falsa #3.** En la nota anterior escribí que subir la constante de exploración no sirve de nada porque multiplicar `0.005` por tres sigue dejándolo en nada. Hmmm, esa es cierta a medias, y la media que falta resulta ser la importante. Volvemos a ella al final, que si no me quedo sin tercer acto.

Lo bueno de escribir las cosas es que quedan escritas y luego puedes comprobarlas. Lo malo es que quedan escritas y luego puedes comprobarlas.

## La cuenta

Un hijo sin visitar en PUCT tiene $Q$ vacío, así que su puntuación entera es el término de exploración,

$$c \cdot P \cdot \sqrt{N}$$

y recibe su primera visita cuando eso supera la puntuación del favorito, que es esencialmente su $Q$. Despejando el presupuesto necesario, tenemos

$$N > \left(\frac{Q}{c\,P}\right)^{2}$$

Hasta aquí no hay nada nuevo, es la fórmula de [[HEX-08 Cuánta intuición hace falta para ser experto]] mirada al revés. Lo interesante empieza cuando en vez de razonar sobre la forma, que es lo que hace todo el mundo incluido yo durante varios meses, metes tus propias constantes.

La mezcla uniforme con $\lambda = 0.4$ sobre `25` casillas le da a cada jugada legal un suelo de

$$\frac{0.4}{25} = 0.016$$

mi constante de exploración valía `1.0`, y una búsqueda de `200` simulaciones tiene $\sqrt{199} \approx 14.1$.

Con lo cual el término de exploración de una casilla que está en el suelo vale

$$1.0 \times 0.016 \times 14.1 = 0.226$$

y ni una milésima más. Ese número, doscientas veintiséis milésimas, es el techo absoluto de lo que mi búsqueda podía dedicarle a una jugada despreciada. Le voy a llamar **el techo**, porque no se me ocurre nada más ingenioso y porque los nombres pomposos para constantes propias los dejo para cuando gane un premio de bellas artes en Zúrich o me inviten a dar una charla TED.

Ahora la otra mitad, que es la que convierte una cuenta en una historia.

Yo llevaba un registro del valor absoluto mediano del value sobre un conjunto fijo de posiciones, que es una forma barata de medir cuánto grita la red. No mide exactamente cuánto sabe, sino que mide cuánto está convencida de lo que cree saber, que es una variable bastante más interesante.

```python
entropy = -(probs * torch.log(probs + 1e-12)).sum(dim=-1)
effective_moves  = torch.exp(entropy).median().item()
value_confidence = values.abs().median().item()
```

En la iteración `0` el value valía `0.2299`. En la iteración `1` valía `0.2905`.

El cruce está ahí. Entre esos dos números, justo encima del techo.

Y si te vas al registro de la política de apertura, en la iteración `0` las veinticinco casillas reciben al menos una visita, y en la `1` veintitrés de ellas caen a **cero**.

Cero punto cero cero cero cero cero cero cero. Como si alguien hubiese borrado la casilla del tablero con Photoshop. Y ahí se quedaron durante las noventa y ocho iteraciones restantes, tan tranquilas, mientras yo miraba la curva de pérdida bajar y me felicitaba y yo me sentía bien.

## Lo que significa

La exploración de mi motor se apagó en la iteración uno. Todo lo demás fue una consecuencia.

Durante el resto del entrenamiento, la búsqueda en la raíz estaba mirando **dos casillas de veinticinco**. Con lo cual el reparto de visitas que servía de objetivo de entrenamiento repartía entre dos casillas, con lo cual la red aprendía a proponer dos casillas, con lo cual la búsqueda seguía mirando dos casillas.

Es, digamos, un tornillo que se aprieta solo, y encima, aprende. Y cada iteración le da otra vuelta.

Por eso 100 iteraciones no arreglaron nada. No le estaba dando más tiempo para aprender, le estaba dando más vueltas al tornillo. Al final del entrenamiento el value mediano estaba en `0.998`, que metido en la fórmula de arriba pide unas `3900` simulaciones para que una casilla del suelo reciba **una** visita. Y sin la mezcla uniforme, con el prior real de la red, el número se te va a nueve cifras.

O sea que el `2225` que me pareció un dato curioso solo era alcanzable gracias al parche que yo había puesto meses antes para otra cosa completamente distinta. Llevaba meses usando la muleta como si fuera una pierna y creyendo que caminaba.

## Un experimento que sí separa cosas

Aquí hay un detalle metodológico del que estoy razonablemente orgulloso, porque es la primera vez en todo el proyecto que diseño un experimento en vez de mirar un número y opinar sobre él.

Si comparo dos entrenamientos y el segundo juega mejor, no sé si es porque la red aprendió más o porque le estoy dando mejores ajustes de búsqueda a la hora de jugar. Son dos cosas distintas y el resultado final las mezcla sin avisarte de nada. Así que entrené dos redes desde cero, con la misma semilla, cambiando **solo** la configuración de self-play. Y luego las examiné a las dos con los ajustes idénticos, apagando el ruido y devolviendo la constante a su valor original.

Mismo examen para las dos. Lo que salga es de las redes y no de la ayuda que les di durante el examen.

| | jugadas efectivas | $\lvert v \rvert$ mediano | casillas visitadas en la raíz |
|---|---|---|---|
| configuración original | `13.4` | `0.53` | `16` de `25` |
| con los arreglos | `16.4` | `0.32` | `25` de `25` |

Seis iteraciones cada una, que es poquísimo, y aun así la separación ya está ahí. Las tres columnas van en la misma dirección a la vez, cosa que con una sola semilla no demuestra absolutamente nada y a la vez apunta bastante. Es lo que hay, tampoco voy a montar un estudio con cincuenta semillas para un tablero de veinticinco casillas.

## Las jugadas efectivas

Esa primera columna merece explicación, porque es la métrica que más he usado desde entonces y que no había necesitado hasta ahora.

Coges la policy de la red en una posición, calculas su entropía, y exponencias.

$$e^{H(p)}$$

Si la red reparte por igual entre `20` casillas legales, la entropía es $\ln 20$ y esto devuelve `20`. Si lo apuesta todo a una, la entropía es cero y esto devuelve `1`. En medio te da un número que se lee directamente como cuántas jugadas se está planteando de verdad, sin tener que traducir nada mentalmente.

Está en [[LEM-entropía]]. Resulta que esto existe desde hace décadas en otro campo con otro nombre (y seguramente en más), porque en ecología lo llaman número efectivo de especies y sirve para distinguir un bosque con diez árboles de diez tipos de otro con noventa robles y un pino desgraciado. Curioso cuanto menos, la verdad.

Mi red vieja acabó en `2.71` jugadas efectivas de unas veinte legales, con un value en `0.998`. Una red sin entrenar da `18` y `0.03`, o sea que considera casi todo y no tiene ni idea de nada, que es lo apropiado para alguien que acaba de nacer. Las dos van a bajar durante el entrenamiento, porque una red que aprende se vuelve más selectiva y más segura, y eso es correcto y deseable. Lo que hay que vigilar es que no se desplomen. Un motor que considera tres jugadas de veinte no es un motor decidido, es un motor con anteojeras.

## Y la tercera cosa falsa

Subir la constante de exploración no sirve de nada.

Es cierta para la casilla concreta que yo estaba mirando aquel día. Multiplicar `0.005` por tres la deja en `0.015` y sigue sin cruzar nada. Y es falsa para todas las demás. Porque una vez tienes la mezcla puesta, ninguna casilla está en `0.005`. Están todas en el suelo de `0.016`. Y ese suelo sí responde a la constante, con lo cual pasar de `1.0` a `2.5` divide el techo por más de seis. O sea que saqué una conclusión mirando un caso particular, la generalicé a todos los casos, y resulta que el caso particular era justamente el único donde la conclusión valía. Eso tiene nombre en estadística y no es un nombre halagador, y me lo he ganado a pulso, y soy tonto.

La ventaja de cometer el mismo error con bastante ruido alrededor es que a la segunda ya suele ser más difícil no verlo.

Vamos a arreglarlo en [[HEX-14 Tres arreglos que no son intercambiables]].

---
title: HEX-06 MCTS
tags:
  - hilo
---
![[mcts-algorithm-1a.png]]

En 1946 Stanisław Ulam estaba convaleciente de una encefalitis y se pasaba las tardes haciendo solitarios, porque no daba para mucho más. En algún momento le entró la duda de cuál era la probabilidad de que un Canfield saliera. Se puso a calcularlo combinatoriamente, como haría cualquier matemático de Los Álamos con tiempo libre, se atascó, y entonces tuvo la idea que nos ocupa, que consistía en que era muchísimo más fácil jugar cien partidas y contar cuántas salían.

Metropolis le puso nombre al método por el casino donde el tío de Ulam pedía dinero prestado para ir a jugar, no jokes.

O sea que el nombre viene de un pariente con problemas de ludopatía y no de ninguna elegante tradición matemática. Muy propio del campo, la verdad.

> [!info] Aquí se habla de árboles (trees) todo el rato. Si no conoces la estructura de datos, esto es prerrequisito y estaría bien que volvieras después de pedirle a la IA que te lo enseñe rápidamente.

## La pregunta

Estoy en una posición y tengo que elegir entre `25` movimientos. Quiero el mejor.

Y aquí, antes de nada, conviene reconocer que la respuesta perfecta existe y es tediosa. Consiste en explorar el árbol completo de continuaciones hasta el final de la partida. Un árbol de juego es literalmente el diagrama de rutas de una novela visual, con la diferencia de que las novelas visuales tienen tres opciones por nodo y aquí hay veinticinco, y con la otra diferencia, más importante, de que en una novela visual alguien dibujó ese diagrama a mano.

El primer movimiento tiene `25` opciones, el segundo `24`, el tercero `23`. Multiplicando sale `25!`, que son aproximadamente $1.5 \times 10^{25}$ caminos.

Muchos acaban antes de llenar el tablero, porque alguien conecta sus bordes y la partida termina. Vale. El número real es bastante menor. Pero igual sigue siendo lo bastante grande como para que la palabra "menor" resulte cómica.

Pues nada, estimamos.

## Idea número uno, jugar al azar

Coges una posición y juegas una partida entera con movimientos aleatorios hasta que alguien gana. Eso se llama **rollout** y no requiere saber absolutamente nada del juego más allá de las reglas, que es exactamente el tipo de requisito que me interesa.

Repites unas cuantas veces y te queda un estimador,

$$V(a) = \frac{W(a)}{N(a)}$$

donde $N(a)$ son las veces que probaste el movimiento $a$ y $W(a)$ las veces que la cosa acabó bien.

A una persona ligeramente buena en mates esto le va a parecer lo normal, porque es la proporción entre lo que has ganado y lo que has jugado. Esto ya funciona, pero cómo no, funciona mal.

Funciona mal porque una continuación aleatoria desde una posición objetivamente ganada se pierde más o menos la mitad de las veces. Tú vas ganando, el generador de números pseudoaleatorios decide colocar tu ficha en la esquina más inútil del tablero, y ahí se acabó tu ventaja. El azar no tiene ningún interés particular en que tú ganes, así que las mates no hacen crecer tu ego per se.

Ahora bien, el estimador no tiene sesgo, así que con suficientes muestras converge al valor real. Lo cual está muy bien en el límite y sirve de bastante poco cuando el presupuesto son trescientas simulaciones en total.

Con lo cual el problema deja de ser estadístico y pasa a ser de reparto, que es un giro que a mí me gustó bastante cuando lo entendí.

## Idea número dos, repartir mejor

Supongamos `300` rollouts.

Repartirlos por igual entre los `25` movimientos da doce cada uno, y doce muestras de un estimador tan ruidoso no distinguen absolutamente nada. Encima estoy gastando doce rollouts en movimientos que se ven malos desde el primer vistazo.

Lo que quiero es lo contrario, o sea gastar la mayoría del presupuesto en los pocos que parecen prometedores y unos pocos en el resto por si me he equivocado.

**Y ese es todo el problema de MCTS.** Pues hala, ya sabes lo que es MCTS y no he tenido que explicarte mucho más. No es un algoritmo de evaluación, es un algoritmo para repartir atención. La evaluación viene de los rollouts, que son tontísimos y no se enteran de nada. Lo listo es a quién se los das.

## Idea número tres, y aquí me equivoqué

La regla obvia que queda es dar el siguiente rollout al movimiento con mejor $V(a)$. Explotar lo que ya sabes.

Falla de una forma ridícula. El primer movimiento que pruebes y salga bien se lleva el segundo rollout, y si vuelve a salir bien se lleva el tercero, y acabas con `297` rollouts en el movimiento que tuvo suerte dos veces seguidas mientras los otros veinticuatro se quedan con uno cada uno. Nunca descubres que el número siete era mejor porque nunca vuelves a mirarlo.

Es el greedy que pasa los tres primeros tests y muere en el catorce, con la diferencia de que aquí nadie te va a dar un veredicto. El programa corre, devuelve un movimiento, y el movimiento es peor de lo que podría haber sido. Fin.

Hace falta un segundo término que premie lo poco explorado.

$$V(a) + \text{algo que crece cuando } N(a) \text{ es pequeño}$$

Yo intenté escribir ese "algo" de memoria varias veces sobre papel y me salió

$$\frac{N(a)}{N_{\text{padre}}}$$

que es la proporción de visitas que se lleva ese hijo. Suena razonable durante unos treinta segundos, hasta que compruebas qué pasa con un hijo visitado una vez de cien, que da `0.01`, frente a uno visitado noventa veces de cien, que da `0.9`.

Vamos, que lo puse al revés. Premiaba exactamente lo contrario de lo que quería premiar.

Y lo bonito, si es que se le puede llamar bonito, es que un término así **también** produce un algoritmo que funciona regular y no casca nunca. Si no llego a hacer la cuenta a mano, me lo trago entero y me quedo tan pancho.

## La fórmula, por fin

La buena viene de un sitio que no tiene nada que ver con juegos de tablero, y está en [[LEM-bandido multibrazo]]. Recomiendo pasar por ahí antes de mirar esto, porque si no parece que alguien lanzó un logaritmo y una raíz cuadrada contra una fracción a ver qué salía.

$$V(a) + c\sqrt{\frac{\ln N_{\text{padre}}}{N(a)}}$$
``` python
def calculate_ucb1(self) -> float:
    exploitation = self.mean_value()
    exploration  = C_PARAM * math.sqrt(math.log(self.parent.visits) / self.visits)
    return exploitation + exploration
```

Se llama **UCB1**, de Upper Confidence Bound, cota superior de confianza. El primer término es lo que has observado y el segundo es el margen de error alrededor de eso. Sumados, no estás comparando medias. Estás comparando el mejor valor que cada opción podría plausiblemente tener.

Y de ahí sale lo que me parece más elegante del algoritmo entero. Un movimiento puede ganar la comparación porque de verdad es bueno, o porque apenas lo has mirado y la incertidumbre está haciendo el trabajo por él. Explotación y exploración salen de la misma regla sin que nadie las escribiera por separado. No hay ninguna línea que diga "y ahora explora un poco".

De la forma solo hay que retener cuatro cosas.

- Las visitas del hijo van **abajo**, porque más información significa menos incertidumbre.
- Las del padre van **arriba**, porque el tiempo revaloriza lo abandonado. Si llevas mil simulaciones y una casilla sigue con dos visitas, esa casilla se vuelve sospechosamente barata de mirar.
- El **logaritmo** hace que ese crecimiento sea lento. Si creciera rápido, la exploración acabaría dominando y no te quedarías nunca con nada.
- La **raíz** sale de una desigualdad de concentración, con el efecto práctico de que el bono decae como $O(1/\sqrt{N})$.

Por cierto, este problema de "repartir un presupuesto limitado entre opciones de rendimiento desconocido" es el que resuelven los sistemas que deciden qué anuncio enseñarte, y el que aparece en cualquier sitio donde alguien tenga que asignar capital sin saber todavía dónde está el retorno. Las gentes que se dedican a eso lo llaman de otras maneras y cobran bastante más que yo por hacerlo. Es la misma fórmula (teóricamente).

## Las cuatro fases

Con la regla en la mano, cada simulación es un viaje de ida y vuelta.

![[mcts.jpg]]

1. **Selección.** Empezamos en la raíz, comparamos sus hijos por UCB1, bajamos al mejor. Repetimos mientras el nodo tenga todos sus hijos creados. Paramos al llegar a uno que no los tiene, o a uno terminal.
2. **Expansión.** Cogemos uno de los movimientos que ese nodo no ha probado y le creamos el hijo. Da igual cuál.
3. **Rollout (o simulación).** Desde el nodo recién creado, una partida aleatoria hasta el final. No guardamos nada, solo quién ganó.
4. **Backup.** Subimos por el camino que recorrimos actualizando en cada nodo el contador de visitas y el valor acumulado.

Repetimos unos cientos de veces y jugamos el movimiento del hijo **más visitado**.

Ahora bien, ¿por qué el más visitado y no el de mejor media? Porque un hijo puede tener media `1.0` con dos visitas, lo cual es suerte y nada más, mientras otro tiene `0.7` con trescientas. El más visitado es donde la búsqueda invirtió más esfuerzo, y para acumular esas visitas ese hijo tuvo que ganar la comparación una y otra vez contra todos los demás.

La media es una muestra. Las visitas son un veredicto repetido trescientas veces.

## El árbol sale torcido y eso está bien

Esta parte me sigue pareciendo la más bonita, y es cuánto emerge de tan poco.

Expandimos un hijo por simulación, nunca todos de golpe. Si expandieras todos, en `5x5` crearías veinticinco hijos, luego veinticuatro por cada uno de esos, y el árbol te explota en la cara antes de llegar a ninguna parte. Uno por simulación significa que con `300` simulaciones tienes `300` nodos como mucho.

Y como la selección solo alcanza las ramas que UCB1 va prefiriendo, los hijos de una rama mala se crean tres o cuatro y ahí se quedan para siempre, mientras que una rama prometedora se va abriendo hacia abajo con una profundidad que no se parece en nada a la de sus hermanas.

Esa asimetría no está programada, y creo que solo se podría ver imaginándotela o con alguien que haya hecho un ejemplo. Hay una fórmula que se aplica igual en todos los nodos y la forma sale de aplicarla muchas veces. Es lo mismo que pasa con un hormiguero, que nadie lo diseñó y todo el mundo asume que sí.

![[mctsasymetrical.png|0]]

El árbol además **persiste entre simulaciones**. Esto no es una búsqueda que empieza de cero cada vez, las trescientas construyen y refinan el mismo árbol, y por eso la simulación quinientas es mucho más lista que la tercera.

## El nodo, y una decisión de la que me arrepiento a medias

Cada nodo guarda cinco cosas y cada una está ahí porque alguna fase la pide.

- El **tablero** de esa posición, porque la expansión necesita saber desde dónde expandir. 
- El **movimiento** que llevó hasta él.
- Un puntero al **padre**, que es lo que permite que el backup sea un `while` en vez de una recursión porque la recursión es mala para la pila, cuándo te vas a enterar de eso, Ulises.
- La lista de **hijos**.
- Y los dos **contadores**.

Lo del tablero tiene discusión, porque hay dos maneras y las dos son defendibles. O guardas una copia en cada nodo, que gasta memoria y te deja consultarlo al instante, o guardas solo el movimiento y reconstruyes la posición bajando desde la raíz cada vez que la necesites.

Elegí la primera. En `5x5` una copia son veinticinco enteros y dos union-find pequeños, con lo cual la memoria no es problema, y un bug de reconstrucción sería exactamente el tipo de fallo silencioso que no quiero tener rondando.

Esa decisión me costó luego una tarde entera de perfilado, que está en [[LEM-deepcopy y perfilado]] y que no vi venir en absoluto. Sigo pensando que elegí bien y aun así me costó una tarde, no lo vi venir y me da igual.

Lo del movimiento parece redundante ahora mismo, ya que el tablero contiene toda la información. Pero creo que lo veremos más adelante.

## Perspectivas

Y ahora la decisión que en su momento me pareció trivial y que acabó siendo la más cara del proyecto.

Cuando el backup sube por el árbol, ¿desde el punto de vista de quién se guarda el valor de un nodo? Una victoria de blancas es buena para los nodos donde mueven blancas y mala para los demás, así que hay que elegir un convenio y sostenerlo en todas partes.

Hay dos que funcionan igual de bien. O guardas el valor desde la perspectiva del jugador que mueve **en** ese nodo, o desde la del que hizo el movimiento **para llegar** ahí.

Elegí la segunda, y el argumento fue que hace que comparar hijos sea inmediato. Cada hijo guarda su valor desde tu punto de vista, así que coges el mayor y ya está, sin negar nada, sin condicionales, sin pensar.

El precio es que la inversión se traslada al backup, donde vas alternando el signo nivel a nivel. Ahí, en una línea que ocupa cuatro caracteres, es donde va a vivir uno de los dos errores que casi me tiran el proyecto. 
```python
def _backup(self, node: Node, value: float) -> None:
    curr = node
    v = value
    while curr is not None:
        curr.visits += 1
        curr.value_sum += v
        v = -v  # <------------ los cuatro caracteres
        curr = curr.parent
```

Está en [[HEX-11 Cuanto mejor, peor]] y todavía queda camino, así que de momento disfrutemos de la sensación de haber tomado una decisión de diseño limpia. Bien, Lester, ¡muy bien!

## Que funciona

Con esto ya hay un motor. Sin nada aprendido, sin redes, sin entrenar absolutamente nada. Solo reglas, rollouts al azar y una fórmula de reparto.

Le puse a jugar `300` partidas contra un agente que elige casillas al azar y ganó las `300`.

Eso no demuestra que sea bueno. De hecho, un rival aleatorio pierde contra prácticamente cualquier cosa que tenga un plan, aunque el plan sea tirar dados con criterio. Lo que demuestra es que no está roto, que es exactamente lo que necesitas saber antes de construir encima, porque a partir de aquí las cosas dejan de fallar de forma ruidosa y empiezan a fallar de la otra manera.

## Los dos agujeros

Aun así, amigos y amigas, esto tiene dos problemas gordos y los dos están en el mismo sitio.

El primero es que **el rollout es malísimo**. Una única partida aleatoria, ruidosa hasta decir basta, y encima cuesta jugar cuarenta movimientos para conseguir un solo bit de información. Cuarenta movimientos para averiguar quién gana en una línea que a nadie le importa.

El segundo es que **al expandir un nodo todos sus hijos empiezan idénticos**. Cero visitas, cero información, y UCB1 no tiene ninguna razón para preferir a ninguno, así que los va probando por turnos hasta que las primeras simulaciones los diferencian. En `5x5` son veinticinco hijos y se aguanta. En `11x11` son 121, y te gastas media búsqueda en el primer nivel antes de haber mirado nada de verdad.

Los dos se arreglan con la misma cosa, y esa cosa es [[HEX-07 Dos cabezas]].
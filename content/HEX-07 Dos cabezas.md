---
title: HEX-07 Dos cabezas
tags:
  - hilo
---
En [[HEX-06 MCTS]] quedaron dos problemas bastante gordos. El rollout aleatorio es un estimador penoso y, cuando expandes un nodo, todos sus hijos son básicamente igual de prometedores para la búsqueda porque todavía no sabe nada de ninguno.

Resulta que una red neuronal puede hacer las dos cosas a la vez, que ya puestos a hacerle trabajar también podemos ahorrarnos una segunda. No dos redes, una sola con dos salidas, y el motivo de que sea una y no dos está al final de la nota porque quiero que llegues ahí habiendo entendido primero qué hace cada salida.

Le pido un **value**, que es un escalar que dice si la posición es favorable para quien mueve, y que va a ocupar el sitio del rollout. Y le pido una **policy**, que es un vector que dice dónde parece estar la jugada buena, digamos que es algo así como la (intuición), y que va a servir para que los hijos recién creados dejen de ser todos iguales. Es, simplificando muchísimo, la intuición de la red sobre dónde merece la pena mirar primero.

Sin la segunda esto no sería AlphaZero, sería un MCTS con evaluación aprendida, idea que ya andaba dando vueltas en los 2000 y que no tiene nada de particular. La policy es la pieza que convierte el asunto en otra cosa.

Recordemos también, por si acaso, que una capa es toda pieza con pesos que pueden aprender y actualizarse mediante backpropagation. Con eso en la cabeza, vamos.

## Lo que ve

La primera capa recibe la matriz de la partida. Las piezas del jugador que mueve van con `1`, las del rival con `-1`, los huecos con `0`. Esto obliga a multiplicar por `-1` cada vez que pasamos de turno, cosa que suena a detalle administrativo y no lo es en absoluto.

Se llama **forma canónica**. La ventaja es que la red solo tiene que aprender un concepto, que es si voy ganando yo, en vez de dos conceptos simétricos que además tendría que mantener sincronizados. Cada partida de entrenamiento vale el doble.

Si quisiera entrenarla en los dos a la vez tendría que meter en algún sitio de la entrada cuál es el jugador, y confiar en que la red descubra sola que ese canal significa "invierte todo lo demás". Que es pedirle bastante y desperdiciar la mitad de su capacidad en aprender una simetría que yo ya conozco.

Hay otro problema, más específico de este juego. En Hex las blancas ganan conectando norte con sur y las negras este con oeste, así que la misma disposición de fichas significa cosas distintas según de quién sea el turno. Para arreglarlo basta con trasponer la matriz además de negarla.

Que eso funcione depende enteramente de que la relación de vecindad sobreviva a la trasposición, cosa que da la casualidad de que es cierta y que está demostrada en [[LEM-forma canónica]].

Y digo "da la casualidad" con toda la intención, porque no lo es. La trasposición resulta ser justamente la simetría que intercambia los papeles de los dos jugadores, y ese es el motivo profundo de que la forma canónica salga exacta y no aproximada. Cuando algo sale demasiado limpio suele ser porque hay una estructura debajo que todavía no has nombrado.

## Los filtros

Una red convolucional está hecha de filtros. Le he puesto `16`. ¿Por qué `16`? Porque es potencia de dos, porque no es un número ridículo para una red tan pequeña, porque se ve bastante based y porque en algún momento hay que dejar de hacer hyperparameter science y escribir un número.

Son matrices de `3x3` que empiezan siendo números aleatorios, se mueven por toda la entrada, y cada una devuelve un mapa del tamaño del tablero.

Los números que guarda cada filtro son los pesos de un patrón. Esta capa inicial mira directamente los `1`, `-1` y `0` crudos, así que solo puede detectar cosas muy básicas. Un filtro puede especializarse en "aquí hay una ficha nuestra aislada", otro en "aquí hay una enemiga", otro simplemente en zonas vacías.

Cuando termina, escupe un bloque de `16` matrices de `5x5`, que se llaman `16` canales. Qué es exactamente un canal está en [[LEM-filtros y canales]], porque si lo meto aquí esto se convierte en un tratado y yo quería escribir una nota.

## ReLU

Antes de pasar al siguiente bloque hay que aplicar una no linealidad, y el motivo se ve mejor construyéndolo.

Una capa convolucional es una operación lineal. Encadeno dos y tengo

$$W_2(W_1(x))$$

que por asociatividad del producto de matrices es

$$(W_2 W_1)x$$

o sea otra operación lineal, la que corresponde a la matriz $W_2 W_1$. Encadeno tres y pasa lo mismo. Encadeno veinte y lo único que consigues es un modelo que tarda veinte veces más en calcular exactamente lo que calculaba con una.

Evidentemente, por supuesto, de nuevo, todos entendemos qué significa esa fórmula, así que pasemos a una más difícil.

$$\max(0, x)$$

Eso es **ReLU**. Y ya está. Ese es el concepto entero, no sé por qué querríamos más. Corta los negativos a cero, deja pasar los positivos tal cual, y con eso la composición deja de ser lineal y las capas de arriba pueden detectar cosas que las de abajo no podían.

Uso esta y no la sigmoid por razones que están en [[LEM-no linealidad]] y que tienen que ver con que la sigmoid estrangula el gradiente por el camino. En una red de tres capas eso casi da igual. En una de treinta te arruina la vida.

## Las capas del medio

La segunda capa recibe el bloque de `16` canales. Como tiene que cruzar toda esa información de golpe, sus filtros ya no son cuadrículas planas sino bloques de `16x3x3`. Al atravesar los canales que escupió la anterior, mezcla los patrones básicos y encuentra relaciones tácticas a corta distancia.

Si la capa inicial marcó en un canal "aquí hay una ficha tuya" y en otro "aquí hay un hueco", un filtro de esta puede cruzar ambas cosas y activarse para decir algo parecido a "esto es un puente potencial".

Sí que es verdad que ojalá los patrones fueran tan fáciles de nombrar. Una red por dentro ve cosas que seguramente nos parecerían alienígenas, porque no tienen ninguna obligación de parecerse a un concepto que un humano haya inventado para hablar de Hex. Otro filtro podría estar detectando "dos fichas enemigas alineadas", y otro podría estar detectando algo para lo que no tenemos palabra y que funciona igual de bien.

La tercera capa hace el mismo proceso sobre los resultados de la segunda. Y aquí aparece una cantidad que hasta ahora no había necesitado.

Una neurona de la primera capa mira una ventana de `3x3` del tablero. Una de la segunda mira una ventana `3x3` de la salida de la primera, y cada casilla de esa ventana ya venía de mirar `3x3`, así que en total abarca `5x5` del original. Una de la tercera abarca `7x7`. En general, para $L$ capas de kernel $k$,

$$R = 1 + L(k-1)$$

y eso se llama **campo receptivo**. Con tres capas de `3x3` sale $R = 7$.

Ahora bien, ¿por qué me importa esto a mí y no le importa a alguien que clasifica fotos de gatos?

Pues porque un gato ocupa una fracción de la foto y le da exactamente igual lo que haya en la esquina de enfrente. Aquí el objetivo es conectar dos bordes opuestos, y una conexión puede depender de lo que pasa al otro lado del tablero. Necesito que alguna neurona sea capaz de verlo entero.

En `5x5` un campo receptivo de `7` va sobrado desde cualquier casilla. **En `11x11` no**, y ahí la profundidad de la red deja de ser una elección más o menos arbitraria y sale de la fórmula. Es una de las pocas veces en este proyecto en las que puedo justificar un hiperparámetro con una cuenta y no con el siempre fiable "lo probé y funcionó". Más en [[LEM-campo receptivo]].

## El flatten y la bifurcación

Cuando la última convolucional termina, tenemos otra vez `16` matrices de `5x5` llenas de lo que me gusta llamar **afirmaciones estratégicas**. Usamos `flatten` para aplastar el bloque y ponerlo en fila, igual que e. n MNIST. En total `400` números.

El `flatten` es exactamente esa operación que hice en MNIST y que entonces parecía trivial porque nadie te estaba intentando matar en un tablero de Hex. Aquí tampoco hace gran cosa; simplemente convierte la red en algo que las capas densas pueden tragarse.

El `flatten`, de nuevo, no es una capa ni tiene pesos. Es una forma de reorganizar los números para que la geometría no se rompa y el backprop pueda armarlo todo de vuelta.

Hasta aquí todo esto es el **tronco compartido**. A partir de ahora la red se bifurca en dos ramas independientes, cada una con sus propias capas densas y sus propios pesos.

Y ahora sí, por qué comparten tronco. Los patrones que sirven para saber quién va ganando son en gran parte los mismos que sirven para saber dónde jugar. Reconocer un muro rival o una conexión a punto de cerrarse hace falta para las dos cosas. Con dos redes separadas, cada una tendría que aprender ese trabajo por su cuenta y estaríamos pagando dos veces exactamente lo mismo, que es un lujo que uno solo se puede permitir cuando le sobran datos. A mí no me sobran, tengo 1500 partidas solamente, y creo que menos.

### Value head

Los `400` números entran en una capa densa que funciona con la misma lógica que la de MNIST. Su trabajo es juzgar toda la lista de tácticas. Si le llega un valor alto del patrón "ruta libre hacia la victoria" lo multiplica por un peso positivo grande, y si el patrón es "el enemigo nos ha cortado el paso" lo multiplica por uno negativo. Se suma todo y listo.

Y aquí hay que parar, porque una capa densa a secas escupe cualquier número real. `-847`, `+6767` o lo que le apetezca.

Recuerda qué hace el MCTS con ese valor. Lo acumula, lo divide entre visitas y lo usa como término de explotación, comparándolo contra un término de exploración que vale del orden de `1` o `2`. Si la explotación puede valer tres mil, el término de exploración deja de existir en la práctica. Da igual lo que valga, no mueve la comparación, y la búsqueda se vuelve completamente codiciosa.

Apunta esto, que dos componentes perfectamente correctos se arruinen mutuamente porque sus escalas no casan. Va a volver más adelante de una forma bastante menos evidente que esta, y cuando vuelva no voy a reconocerlo hasta tres días después.

La segunda razón es que el value se entrena contra resultados de partida, que son `+1` o `-1`, con lo cual una red capaz de escupir `6767` está prediciendo algo que no ha ocurrido nunca ni va a ocurrir. O ganas o pierdes. No es que ganes seis mil setecientas sesenta y siete veces.

Por eso al final va **tanh**, que acota la salida a $(-1, 1)$, que es exactamente el rango de las etiquetas, con el cero significando posición equilibrada. Está en [[LEM-tanh]].

```python
# el tronco termina y se bifurca
policy_logits = self.policy_layer(flat)
value = torch.tanh(self.value_layer(flat)) # tanh está acotado a [-1, 1]
```

### Policy head

Los mismos `400` números entran en otra capa densa que saca un vector de tamaño igual al número de casillas, `25` en `5x5`. Al final va **softmax** para que sea una distribución de probabilidad que sume `1`.

Con la particularidad de que hay que tapar las casillas ocupadas antes. Si no lo haces, la red reparte probabilidad entre sitios donde no se puede jugar, y esa probabilidad es probabilidad que no se llevan las casillas legales. Se resuelve con un truco pequeño que tiene su gracia y está en [[LEM-softmax enmascarado]].

```python
valid = torch.zeros(total_cells, dtype=torch.bool)
valid[node.board.valid_choices()] = True

# tapar las casillas ocupadas antes de normalizar
policy_logits[~valid] = -torch.inf 
policy = torch.softmax(policy_logits, dim=0)
```

Este vector es lo que va a hacer que los hijos recién creados dejen de ser indistinguibles. Cómo entra exactamente en la búsqueda es la nota siguiente, y ahí hay más miga de la que parece.

## Y ya está


![[forwardnetwork.jpg]]

La red entera cabe en unas 30 líneas de PyTorch.

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class PolicyValueNetwork(nn.Module):
    def __init__(self, board_size: int = 5):
        super().__init__()
        self.board_size = board_size
        self.initial_conv = nn.Conv2d(in_channels=1, out_channels=16, kernel_size=3, padding=1)
        self.convolutional_1 = nn.Conv2d(in_channels=16, out_channels=16, kernel_size=3, padding=1)
        self.convolutional_2 = nn.Conv2d(in_channels=16, out_channels=16, kernel_size=3, padding=1)

        flat_features = 16 * board_size * board_size
        self.value_layer = nn.Linear(in_features=flat_features, out_features=1)
        self.policy_layer = nn.Linear(in_features=flat_features, out_features=board_size*board_size)


    def forward(self, x: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]:
        x = torch.relu(self.initial_conv(x))
        x = torch.relu(self.convolutional_1(x))
        x = torch.relu(self.convolutional_2(x))

        flat = torch.flatten(x, start_dim=1)

        policy_logits = self.policy_layer(flat)
        value = torch.tanh(self.value_layer(flat))

        return (policy_logits, value)

```


Después de cuatro semanas leyendo sobre convoluciones, campos receptivos y por qué la sigmoid es mala idea, ver que todo eso son treinta líneas produce una sensación que ya conocía. En competitiva pasa igual, que te tiras cuarenta minutos entendiendo por qué el problema se reduce a un flujo máximo y luego el código son quince líneas de una plantilla que te sabes de memoria.

La parte difícil casi nunca es la que ocupa sitio, como suele pasar. Aquí es donde metemos el hueco de la ducha que habíamos dicho antes.

Lo que no sabía todavía, mientras escribía estas treinta líneas tan contento conmigo mismo, es que la pieza de la que iban a venir absolutamente todos mis problemas era la policy. Y no precisamente por estar mal implementada.

Qué tonto soy.
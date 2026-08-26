---
title: HEX-05 La arquitectura de la cosa
tags:
  - hilo
---


Antes de escribir la primera línea de código hay cuatro semanas en las que no escribo absolutamente nada.

Yo tenía un plan de verano que consistía en tener esto funcionando para agosto, y en agosto seguía leyendo sobre campos receptivos, cosa que a par de amigos le costó bastante entender porque desde fuera parece que no estás haciendo nada. Desde dentro también, la verdad.

Lo que quiero contar aquí es por qué ese mes existió, y de paso responder a tres preguntas que me hice al principio y que en su momento no supe contestar.

- ¿Por qué hacen falta dos piezas y no una?
- Si una es lenta y la otra rápida, ¿por qué la lenta le enseña a la rápida y no al revés?
- ¿Cómo sé que esto no se atasca en un bucle de tonterías retroalimentadas?

Las dos primeras las contesto abajo. La tercera me llevó bastante más y está en [[HEX-09 Self-play]], porque hay que construir unas cuantas cosas antes de que la respuesta signifique algo.

## Las cuatro cosas que no tienen nada que ver entre sí

PyTorch, que ya está en [[HEX-02 Memoria episódica sin estados intermedios]] y es la más mecánica del grupo. Convoluciones, que vienen de visión por computador y traen su propio vocabulario. Búsqueda en árbol por Monte Carlo, que viene de teoría de juegos. Y aprendizaje por refuerzo, que es donde mis intuiciones estaban más seguras de sí mismas y más equivocadas, combinación que ya me habría gustado detectar antes.

Cada una tiene exactamente el mismo momento, que es que lees la explicación, asientes, cierras la pestaña con la satisfacción del deber cumplido, y luego te sientas a escribirlo o a pensarlo en la ducha y descubres que no lo tenías.

Ese momento merece un nombre, porque me pasó como seis veces y volverá a salir en este cuaderno. Lo voy a llamar **el hueco de la ducha**.

## Cómo programar sin programar

Hay dos escuelas. Aprenderlo todo antes de empezar, que es lo que hice, o empezar a picar y aprender lo que haga falta cuando haga falta, que es lo que hace casi todo el mundo y probablemente por buenas razones.

Elegí lo primero porque no quería estar a mitad de camino descubriendo un concepto que me obligara a reescribir la mitad del proyecto. En ese sentido funcionó, porque efectivamente no tuve que reescribir la mitad de nada.

Lo que no funcionó es que hay conocimiento que **no se puede adquirir en abstracto**. Estuve un rato considerable peleándome con la fórmula de UCB1 sobre la pizarra, reconstruyéndola de memoria o casi de memoria. Luego la implementé, imprimí las visitas de cada nodo, vi el reparto con mis propios ojos, y quedó clara en cinco minutos.

Leyendo no me acercaba tanto, lo cual me hace criticar las formas que tiene la academia y la universidad pública hoy en día para dar clases, joder, si es que no quiero criticar porque si no me sale mal.

En Kingdom Come Deliverance II, al principio, el juego no te explica prácticamente nada y hay una barrera donde parece que lo estás haciendo todo mal. Y lo estás haciendo mal, efectivamente. Sin embargo, ninguna cantidad de leer sobre el mundo sustituye a meterte en él, y sospecho que el juego lo sabe y que por eso no te lo explica.

O sea que la respuesta correcta es una mezcla y yo me pasé de frenada por el lado teórico. Dejémoslo ahí, que tampoco vamos a psicoanalizarme en la 5th nota.

## Pregunta uno, por qué dos piezas

Empecemos por lo más tonto posible y vamos subiendo.

Imagina en este momento que solo tienes **la red**. Le enseñas un tablero y te dice dónde jugar, al instante. El problema es que solo es tan buena como lo que haya aprendido, y al principio no ha aprendido nada, con lo cual juega como si le hubieras dado un mando a un gato.

Vale, pues imagina lo contrario, que solo tienes **la búsqueda**. Explora cientos de continuaciones desde la posición actual y con eso se hace una idea razonable de qué jugada es buena. Funciona desde el minuto cero sin haber aprendido nada, porque las reglas ya están ahí y las posiciones terminales no admiten discusión.

Y aquí es donde uno se pregunta, con toda la razón, para qué demonios querría entonces la red. Si la búsqueda funciona sola, quítale la red y a correr.

Pues porque la búsqueda es lenta y ciega. Reparte su atención sin ninguna preferencia, así que gasta lo mismo en la jugada que gana la partida que en la que la tira. Una jugada de veinticinco, sin información previa, se lleva una veinticincoava parte del presupuesto y ni un céntimo más. Es igual que el bucle cuadrático en $O(N^2)$ que te da la respuesta correcta pero te da TLE.

La red, en cambio, es instantánea y está sesgada, que aquí resulta ser una virtud. Si la red dice "mira por aquí", la búsqueda mira por ahí primero y gasta el presupuesto donde importa.

Con lo cual las dos piezas no son redundantes. Una aporta cálculo y la otra aporta dirección, y ninguna tiene lo de la otra, porque son como un vector, con su módulo y su dirección.

```python
# network.py
class PolicyValueNetwork(nn.Module):
    def __init__(self, board_size: int = 5):
        super().__init__()
        # tronco común (extrae patrones espaciales)
        self.initial_conv = nn.Conv2d(1, 16, kernel_size=3, padding=1)
        self.conv1 = nn.Conv2d(16, 16, kernel_size=3, padding=1)
        self.conv2 = nn.Conv2d(16, 16, kernel_size=3, padding=1)

        # dos cabezas
        flat_features = 16 * board_size * board_size
        self.policy_layer = nn.Linear(flat_features, board_size**2) # Dirección
        self.value_layer = nn.Linear(flat_features, 1)              # Evaluación

    def forward(self, x):
        # ... paso por las convoluciones ...
        policy_logits = self.policy_layer(flat)
        value = torch.tanh(self.value_layer(flat)) # tanh para acotar entre -1 y 1
        return policy_logits, value
        
```


## Pregunta dos, por qué la lenta enseña a la rápida

Esta tiene una respuesta de una línea y merece la pena verla venir despacio.

**La búsqueda siempre juega mejor que la red que la guía.**

Piénsalo con una persona. Alguien que calcula variantes durante diez minutos juega mejor que ese mismo alguien decidiendo en un segundo. No un poco mejor, sistemáticamente mejor, sin excepciones, mientras calcule bien.

Ahora fíjate en la palabra "mismo". Es la misma persona. Es la misma red. La búsqueda no tiene ninguna fuente de conocimiento adicional, usa exactamente la intuición de la red, y aun así llega más lejos porque ha gastado tiempo.

Entonces la búsqueda sirve de profesor. Entrenas a la red para que prediga de golpe lo que la búsqueda descubre pensando, y, como la búsqueda está dirigida por la red, una intuición mejor produce búsquedas mejores, que producen mejores lecciones, que producen mejor intuición.

Es una escalera que se construye a sí misma mientras la subes. Llevo semanas con esto y me sigue pareciendo mínimamente irrazonable que funcione, y sospecho que esa sensación no se me va a quitar aunque termine el proyecto.

## El plan, y un dilema que no sé resolver

El plan tiene tres fases y cada una produce algo que existe por sí solo. Si me atasco en la segunda no habré perdido el tiempo, porque tendré un motor jugable para enseñárselo a mi familia, que tanto sabe de lo que estoy haciendo.

1. El juego y una búsqueda sin red, con simulaciones aleatorias. Al final hay un bot, no una IA, que toma decisiones razonables. Todo algorítmico, terreno conocido. Así funcionaban las cosas antes de 2016, si no me equivoco; es lo que se le ocurre a cualquier persona que haya visto al bot del tres en raya.
2. La red con sus dos salidas, integrada en la búsqueda para reemplazar las simulaciones aleatorias. Al final de esto no hay nada entrenado todavía, solo la verificación de que las piezas encajan.
3. El bucle de partidas contra sí mismo. Aquí es donde aprende de verdad y donde el depurado se vuelve opaco, porque el síntoma de que algo va mal es que la curva no sube, y eso no te dice absolutamente nada sobre dónde mirar.

Escribo esto sabiendo que la tercera va a ser la larga. Cuánto exactamente, ni idea. Pero me voy a liar lo suyo, estoy seguro.

Y hay algo que me ronda desde que dibujé estas tres fases. La fase uno produce un bot que escribí yo con reglas que yo entiendo. La fase tres produce un motor opaco que aprendió solo. Puedo enfrentarlos, evidentemente.

Pero si el opaco gana, ¿qué he demostrado? Que aprendió algo, o que mi bot era peor de lo que yo creía. Y si pierde, ¿qué he demostrado? Que el aprendizaje no funciona, o que mi bot era mejor de lo que yo creía. Las cuatro formas de ver esto son compatibles con cualquier resultado, y yo soy el autor de las dos partes.

No tengo respuesta. Anoto la duda y sigo, porque la alternativa es no empezar, y esta nota ya lleva demasiadas semanas de no empezar acumuladas.

## Una regla que me impongo

El hueco de la ducha no aparece cuando lo dejas. Aparece tres semanas más tarde, cuando ya has construido encima y ya ni te acuerdas de qué parte era la que no habías terminado de entender.

Así que escribiré aquí las cosas para que no se me olviden, y escribiré de más para que se queden sin olvidarse. Si estás leyendo esto y hay párrafos que te sobran, te sobran a ti, si eso. A mí dentro de dos meses probablemente no.
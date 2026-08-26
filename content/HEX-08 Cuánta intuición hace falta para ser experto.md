---
title: HEX-08 Cuánta intuición hace falta para ser experto
tags:
  - hilo
---
Tengo un árbol que reparte atención y una red con dos salidas. Ahora hay que enchufarlas, que es donde MCTS deja de ser MCTS y empieza a ser un bebé grande tatuado en el trasero de **Juan Luis Nash**.

Y aquí hay una pregunta que voy a dejar planteada y que respondo al final, porque es la que de verdad importa y porque si la contesto ahora no la vas a creer.

Si la red se equivoca con mucha convicción sobre una jugada, ¿puede la búsqueda pasarle por encima?

Es una pregunta bastante importante. Sobre todo porque una red que se equivoca con seguridad es bastante más peligrosa que una que dice "no sé". Esto último también aplica fuera de Hex, aunque no siempre parezca estar de moda.

Guárdala. Volvemos a ella.

## El primer agujero, tapado

El rollout aleatorio se va a tomar por saco.

Cuando la búsqueda llega a una posición que no ha expandido antes, consulta la red una vez y recibe el value. Ese número ocupa el sitio de la partida aleatoria, y es mejor por dos motivos que, de hecho, **no** tienen nada que ver entre sí. Es más rápido, porque una pasada de red frente a cuarenta movimientos al azar no hay color. Y por supuesto es MUCHÍSIMO menos ruidoso, que es lo que de verdad importa, porque una partida aleatoria desde una posición ganada se pierde la mitad de las veces y como estimador eso es un desastre con nombres y apellidos.

## El segundo, y aquí hay que pensar un rato

La policy tiene que hacer que los hijos recién creados dejen de ser **indistinguibles**. La pregunta es dónde se mete exactamente, y conviene descartar el sitio equivocado antes de encontrar el bueno.

En el término de explotación no puede ser. Ese término mide lo que has observado, y de un hijo sin visitar no has observado nada. Meter ahí el prior, entonces, sería mentir sobre la procedencia de la información, que es pecado leve en un examen y bastante más grave en un sistema que se entrena a sí mismo con sus propias conclusiones. Aquí no hay profesor que te mire mal y siga con el siguiente ejercicio, porque no hay ningún problema en que el error se reproduzca, aprenda y luego vuelva en su arco de redención con muchísima más confianza.

Así que va en el de exploración, multiplicando, para que el bono de un hijo sea proporcional a lo prometedor que la red lo considera. Un hijo con probabilidad `0.4` recibe cuatro veces más empujón que uno con `0.1`. Hasta aquí, bien, es razonable.

Pero hay un problema de *forma*. UCB1 lleva las visitas del hijo dentro de la raíz y en el denominador, así que con cero visitas revienta por división entre cero. Antes eso no ocurría nunca, porque solo comparábamos hijos cuando todos tenían al menos una visita. Ahora sí queremos compararlos vacíos, precisamente porque la red ya nos dice algo de ellos antes de mirarlos.

La modificación mínima consiste en **sacar las visitas de la raíz y sumarles uno**.

$$\frac{\sqrt{N(s)}}{1 + N(s,a)}$$

Con cero visitas eso vale $\sqrt{N(s)}$ y no divide entre nada. Conforme las visitas crecen, el bono decae. Mismo comportamiento cualitativo, sin la singularidad.

Juntándolo todo queda **PUCT**, donde la P es de _predictor_ y el predictor es la policy.

$$Q(s,a) + c \cdot P(s,a) \cdot \frac{\sqrt{N(s)}}{1 + N(s,a)}$$
```python
def calculate_puct(self) -> float:
    exploitation = self.mean_value() if self.visits > 0 else 0.0
    exploration  = C_PARAM * self.prior * math.sqrt(self.parent.visits) / (1 + self.visits)
    return exploitation + exploration
```

AlphaZero además quita el logaritmo que UCB1 llevaba dentro de la raíz, y esa parte es empírica y no tengo una justificación bonita que dar. Leí que el efecto es que la exploración crece más deprisa con el tiempo total, lo cual compensa que la policy concentra mucho la búsqueda. Si alguien tiene una derivación decente de por qué, escríbeme y la meto aquí con crédito y con abrazos, porque no logré encontrar una buena.

## La forma, que dice más que los símbolos

Vamos a leer esa expresión en los extremos, porque es donde se entiende.

Cuando un hijo tiene cero visitas, $Q$ está vacío y el prior es la única información que existe en el universo. El prior decide solo.

Conforme las visitas se acumulan, el denominador crece, la contribución del prior se encoge como $O(1/N)$, y la media empírica va tomando el mando y teniendo poder.

O sea que **la intuición se disuelve al ritmo al que la búsqueda averigua cosas por su cuenta**. Es más o menos lo que le pasa a cualquiera que aprende un oficio, que empieza haciendo caso a lo que le dijeron y acaba fiándose de lo que ha visto, y no hay un día concreto en el que cambie de una cosa a la otra. Cosas de la expertise.

![[symbolic_intuition.jpg]]

## Lo que cambia en el ciclo

Meter la red no es cambiar una función, cambia la forma de las cuatro fases.

En el MCTS clásico expandes un hijo por simulación, porque los rollouts te dan información de uno en uno y no tiene sentido crear hijos que no vas a poder evaluar. Aquí la red te escupe las `25` probabilidades de una tacada, así que lo natural es expandir **todos** los hijos legales a la vez y repartirles el prior.

Con lo cual la lista de movimientos por probar deja de tener sentido, porque nunca hay un estado intermedio. Un nodo pasa de no tener hijos a tenerlos todos en un paso, y la condición de parada de la selección cambia, porque ya no paras cuando faltan hijos por probar sino cuando el nodo no tiene hijos en absoluto.

```python
# 25 priors de una tacada y se expande todo a la vez
for choice in choices:
    new_node = Node(board=new_board, prior=policy[choice].item(), move=choice, parent=node)
    node.children.append(new_node)
```

Y las cuatro fases pasan a ser tres. Seleccionas, expandes recibiendo el value, y haces backup. La simulación desaparece del código, lo cual tiene su gracia, porque acabas de quitar la pieza que le daba nombre al algoritmo y el algoritmo funciona mejor que antes. Irónico cuanto menos.

## El caso terminal

Hay uno que resolví mal en el primer intento y que conviene señalar porque va a tener consecuencias.

Cuando la selección llega a un nodo terminal no hay nada que expandir ni nada que evaluar, porque el resultado ya se sabe con certeza. Y con la convención de perspectiva que elegí en [[HEX-06 MCTS]], el valor de un nodo terminal es siempre `+1`, sin condicionales, porque en Hex quien movió para llegar a un nodo terminal es siempre quien acaba de ganar.

Esa **constante inocente** no lo es tanto.

## La constante

El `c` de la fórmula es uno de esos hiperparámetros famosos y yo lo puse a $\sqrt{2}$, porque es el valor canónico de UCB1 y me pareció que se heredaba.

Cosa que resulta que no.

Míralo bien y lento. En UCB1, el término de exploración es la raíz entera, un número del orden de la unidad. En PUCT, ese mismo término va multiplicado por un prior, que es una probabilidad, o sea un número entre cero y uno y normalmente bastante más cerca del cero.

Si el prior típico ronda `0.1`, todo tu término de exploración se ha dividido por diez respecto a lo que valía en UCB1, y la constante tendría que compensarlo. En PUCT la gente suele andar entre `1` y `2.5`. Yo lo bajé a `1.0` bastante después, cuando ya llevaba un entrenamiento entero hecho y guardado, y ahí lo dejé porque me pareció que estaba en el rango razonable y me fui a hacer otras cosas.

Y ahora la moraleja, chicos y chicas.

Una constante fuera de escala **no produce ningún error**. No lanza una excepción, no rompe un test, no deja rastro en ningún log y si rompe cosas, no sabes ni qué rompe ni por qué, aparentemente, a priori. Produce un motor un poco peor. Y "un poco peor" no se distingue de "así es como es" a menos que tengas contra qué compararlo, cosa que en un sistema que solo se mide contra sí mismo es un problema sin solución desde dentro.

Es la versión algorítmica de tener una discusión contigo mismo y declararte ganador, y luego también la versión filosófica de aprender a ganarte a ti mismo jugando al ajedrez. Le pregunté a una amiga recientemente y me dice que es posible, quizás el mundo está roto y yo no lo sabía.

Anyways, esa `c`, esa constante concreta, esa línea, acaba siendo una de las tres cosas responsables de que este motor jugara peor de lo que podía durante un entrenamiento entero. Lo cuento cuando llegue.

## La pregunta del principio

**¿Puede la búsqueda pasarle por encima a una red que se equivoca con convicción?**

Según lo que acabo de explicar, **sí**. La contribución del prior decae como $O(1/N)$ y la media empírica toma el mando, así que basta con darle simulaciones suficientes y la verdad acaba emergiendo.

Eso es lo que yo escribí en su momento y me quedé tan tranquilo. Es la respuesta que da todo el mundo y es la que está implícita en cómo se presenta la fórmula.

Y es falsa, o por lo menos es falsa en el sentido que a una persona normal le importa. El mecanismo que lo impide es exactamente el que acabo de describir con tanta alegría, y para verlo hay que meter en la fórmula constantes concretas en vez de razonar sobre la forma.

En abstracto, la red pierde importancia. En números reales, puede que ni siquiera llegue a mirar la jugada.

Está en [[HEX-13 Doscientas veintiséis milésimas]] y es lo que más he aprendido en todo el proyecto. No lo vi venir ni de lejos, y llevaba escrito en esta misma nota, arriba, desde el principio.
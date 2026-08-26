---
title: HEX-09 Self-play
tags:
  - hilo
---
![[selfplay.jpg]]

Voy a hacer de abogado del diablo contra mi propio proyecto durante una nota entera, porque la objeción que me voy a poner es buena y me la pusieron dos personas distintas antes de que yo supiera contestarla.

La objeción es esta.

*Tu red empieza siendo mierda. La búsqueda que la usa también es mierda, porque está guiada por ella. Los datos que salen de esa búsqueda son una mierda. Entrenas con datos de mierda, obtienes una red un poco distinta pero igual de mierda, y repites. Has construido una máquina de reciclar tus propios errores con pasos intermedios para que parezca ciencia y algo guay para tu currículum. Así ni te van a ver ni te va a apoyar nadie, porque es falso y a la vez infalsable, justo lo que necesita alguien deleznable que solo quiere que lo mimen.*

Es un argumento bueno y se parece a lo que en estadística llaman levantarse **tirando de los cordones de los zapatos**, y suena tan razonable que durante unos días yo mismo no supe por dónde atacarlo.

Hey, pero sin embargo el bucle funciona. Vamos a ver por dónde entra la grieta.

## La única cosa verdadera

Empecemos por lo único de todo el sistema que **no** sale de la red.

Juegas una partida hasta el final. Alguien gana. Eso es un hecho sobre las reglas del juego y no lo opina nadie.

Cada posición donde el eventual ganador era el que movía se etiqueta con `+1`, y las demás con `-1`. Toda la partida se etiqueta de golpe cuando termina, hacia atrás, alternando el signo nivel a nivel. Fíjate en que ninguna de esas etiquetas necesita que la red haya acertado nada.

Ahí está tu grieta. Es pequeña, es una gota, y es auténtica.
Es una cantidad microscópica de verdad, pero por ahora tendremos que conformarnos. El presupuesto de realidad del proyecto no da para más.

## Y el resto sí sale de la red, pero

El policy head es otra historia, porque para entrenarlo de la forma supervisada normal necesitarías saber cuál es el movimiento correcto en una posición. Y nadie sabe cuál es el movimiento correcto en Hex. No existe el dataset y no hay forma de fabricarlo y, cómo no, para fabricarlo necesitarías a alguien que juegue perfecto, que es justamente lo que estoy intentando construir, y por ende, muy bien, bonito bucle y menuda shit. Magnífico.

Pues resulta que la búsqueda ya lo ha producido y encima gratis.

Cuando el MCTS termina sus simulaciones desde una posición, las visitas no están repartidas por igual entre los hijos, se han amontonado sobre los movimientos que la búsqueda encontró prometedores. Normaliza esos contadores,

$$\pi(a) = \frac{N(s,a)}{\sum_b N(s,b)}$$
```python
def _get_pi(self) -> np.ndarray:
    total_visits = sum(node.visits for node in self.root.children)
    pi = np.zeros(self.root.board.board_size ** 2)
    for child in self.root.children:
        pi[child.move] = child.visits
    return pi / total_visits
```

y tienes una distribución sobre exactamente las mismas `25` casillas que escupe el policy head. Se pueden comparar sin ninguna adaptación.

Ahora bien, el abogado del diablo diría que ese $\pi$ salió de una búsqueda guiada por la red, así que es la red opinando sobre sí misma. 

Y tiene razón. Parcialmente. Porque ese $\pi$ **es mejor que la propia intuición de la red**, ya que lleva unos cientos de simulaciones de cálculo real metidas dentro que la intuición cruda nunca tuvo, y parte de esas simulaciones tocaron posiciones terminales, donde la gota de verdad de la sección anterior se cuela hacia arriba por el árbol.

O sea que el bucle no es red hacia red. Es red hacia búsqueda hacia red, con las reglas del juego metiendo información en el medio.

## Dato que a mí me convenció

Mi MCTS con una red **completamente aleatoria**, sin entrenar, pesos recién inicializados, gana el `90%` de las partidas contra un rival que juega al azar.

¿Cómo? La red no sabe nada. La red es ruido.

Porque la búsqueda toca posiciones terminales y ahí el resultado no es opinión de nadie. Ese margen del noventa por ciento no viene de la red, viene de las reglas, porque cuando ganas, ganas, y cuando pierdes, pierdes, y cuando eso pasa se lo dices a tu padre para que llame al profesor (buena referencia). Ese margen es la primera lección que hay para aprender.

Entrenas con ella, la red mejora un pelo, la búsqueda mejora un poco más, y el ciclo se amplifica. Lo que hace que despegue no es que sea listo al principio. Es que **la diferencia entre buscar y no buscar nunca desaparece**.

## La pérdida, y una cosa que hay que grabarse

Las dos cabezas se entrenan con un solo número para que un único `backward()` alcance las dos ramas.

$$L = (z - v)^2 + \left(-\pi^\top \log p\right)$$

Error cuadrático a la izquierda contra el resultado de la partida, entropía cruzada (cross-entropy) a la derecha entre la policy de la red y la distribución de visitas.

Se suman y no se multiplican porque cálculo nos enseñó que la derivada de una suma es la suma de las derivadas, así que cada término aporta su gradiente de forma independiente. Con una multiplicación el gradiente de cada término quedaría escalado por el valor del otro, y una pérdida cerca de cero anularía la contribución de la otra rama entera. Tendrías una red que deja de aprender a jugar porque ha aprendido a evaluar.

Ahora la parte que hay que grabarse, porque más adelante decide si sé leer mis propias gráficas o no.

Una entropía cruzada medida contra una distribución **no puede bajar nunca de la entropía de esa distribución**. Esto es,

$$H(\pi, p) \ge H(\pi)$$

con igualdad solo cuando $p = \pi$.

Una policy completamente plana sobre `25` casillas tiene $H = \ln 25 \approx 3.22$, así que durante las primeras iteraciones, cuando la búsqueda reparte las visitas por igual, la pérdida es **físicamente incapaz** de bajar de ese número por muy bien que aprenda la red. Verla caer por debajo de `3.22` más adelante significa otra cosa, que es que las etiquetas se han afilado. Está en [[LEM-entropía]].

Y date la vuelta a eso un momento. Si el suelo depende de la entropía del objetivo, entonces **la pérdida puede subir mientras el motor mejora**. Basta con que el objetivo se... ensanche.

Escribo esto ahora y suena a nota al pie. Ojalá me hubiera hecho caso a mí mismo.

## Cosas que si no pones nadie te avisa

Tres, y las tres son de una línea.

**Barajar.** Los ejemplos vienen ordenados por partida y dentro de cada partida los valores alternan `+1`, `-1`, `+1`. Si haces lotes en ese orden, cada lote contiene posiciones correlacionadísimas de la misma partida y el gradiente no representa nada más que esa partida. Se barajan antes de partir.

**Simetría.** Si giras el rombo `180` grados tienes el mismo juego. Norte y sur se intercambian, este y oeste también, y ninguno de los dos jugadores cambia de objetivo. En índices eso manda $(r,c)$ a $(n-1-r, n-1-c)$, o sea que niega cada desplazamiento del conjunto de vecinos, y ese conjunto es cerrado bajo negación. Cada posición vale por dos, rotando el tablero y el $\pi$ igual, sin tocar el value. Datos duplicados multiplicando el coste por cero.

```python
# misma posición rotada 180º
augmented.append((
    board[::-1, ::-1].copy(), 
    pi.reshape(5, 5)[::-1, ::-1].reshape(-1).copy(), 
    z
))
```

**No tirar los datos.** La versión ingenua genera cincuenta partidas, entrena con ellas, y las tira. Otra vez cincuenta, entrena, tira. Fíjate en el verbo. Las tira. Cada ronda aprende de un puñado pequeño de partidas correlacionadas, se ajusta a ellas, y pierde el acceso a todo lo anterior. Guardar las últimas diez tandas y entrenar sobre la unión no añade ni un dato nuevo, lo único que hace es que la distribución sea más ancha y cambie más despacio.

Con una pega que es que si dejas las mismas épocas con diez veces más datos has multiplicado por diez el entrenamiento por iteración sin darte cuenta. Así que cuando metes el buffer, bajas las épocas. Makes sense.

## Temperatura

Y aquí el detalle que en su momento me pareció menor.

Mi self-play elige siempre el movimiento más visitado. Con la misma red, todas las partidas contra sí misma salen **idénticas**. Ni una fuente de azar en todo el ciclo. Cincuenta copias de la misma partida, veinticinco posiciones repetidas cincuenta veces, que es la definición operativa de perder el tiempo con estilo.

La solución es muestrear. En vez del máximo, eliges el movimiento según la distribución $\pi$, así que uno con `0.6` se juega el sesenta por ciento de las veces.

Pero muestrear toda la partida mete ruido en el final, donde las jugadas son forzadas y tirar una partida por capricho produce una etiqueta de value que es directamente falsa. Le estarías enseñando a la red que una posición ganada se pierde. Así que se muestrea solo durante los primeros movimientos y luego se pasa al máximo, cosa que está en [[LEM-temperatura]].

Yo puse de primeras `8` de veinticinco porque vino C-Melman y me lo dijo al oído. Realmente no hay mucho más misterio.

Luego acabé subiéndolo a `14`. Y todavía no era suficiente, y el motivo de que no fuera suficiente es más profundo que el número, y tiene su propia nota.

## El bucle entero

1. Jugar una partida contra uno mismo, corriendo MCTS en cada posición, guardando el tablero y el reparto de visitas.
2. Al acabar, etiquetar todas esas posiciones con el resultado.
3. Repetir unas cuantas partidas.
4. Entrenar la red con el contenido del buffer.
5. Volver al 1 con la red nueva.

Le puse 30 iteraciones con 50 partidas cada una. El self-play se lleva prácticamente todo el reloj, y son 50 partidas independientes que no se hablan entre ellas, así que se reparten entre procesos sin ninguna coordinación y el tiempo se divide casi por el número de núcleos. Porque sí, meter aquí multi-processing no está mal.

El entrenamiento en sí cuesta menos de un segundo por iteración.

## Y el abogado del diablo

Vuelvo a la objeción del principio, que era que esto es una máquina de reciclar errores.

La gracia no es que la red empiece sabiendo jugar porque es que NO sabe absolutamente nada. La gracia es que no necesita empezar sabiendo mucho más que la diferencia entre "he ganado" y "he perdido". A partir de ahí, el resto se lo puede ir inventando. A veces bastante bien. A veces con una confianza que, como veremos, habría sido mejor no darle, porque se empieza a creer cosas y empieza a simular el comportamiento de un humano de a pie que ha escrito un libro y basa su identidad en ello para autoentregarse poder a sí mismo.

La respuesta corta a lo primero del abogado es que hay una fuente externa de información, que son las reglas del juego, y entra por las posiciones terminales. La búsqueda las toca, la gota sube por el árbol, y por eso la búsqueda siempre sabe un poquito más que la red que la guía.

La respuesta larga es que esa grieta quizás pueda cerrarse con otra cosa, bien sea avanzada o no. Pero eso es más adelante.



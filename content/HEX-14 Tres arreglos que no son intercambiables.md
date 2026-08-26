---
title: HEX-14 Tres arreglos que no son intercambiables
tags:
  - hilo
---
Sabiendo dónde está el problema toca arreglarlo. Voy a cambiar, por qué no, varias cosas a la vez.

Véase primero [[LEM-camino-minimo-en-hex]], [[LEM-replay-buffer]] y [[LEM-ruido-de-dirichlet]] si se quiere entender.

## Arreglo 1

La mezcla uniforme reparte $\lambda = 0.4$ a partes iguales entre las `25` casillas, con lo cual a cada una le tocan `0.016`. Y ya sabemos, por [[HEX-13 Doscientas veintiséis milésimas]], que `0.016` no cruza el techo cuando el value está gritando.

El **ruido Dirichlet** reparte esa misma masa de forma desigual y distinta en cada partida. En vez de darle una migaja a todo el mundo, le da un trozo decente a dos o tres casillas sorteadas y prácticamente nada al resto. Cosa que está bien.

$$\tilde{P}(a) = (1-\lambda)\,P(a) + \lambda\,\eta_a, \qquad \eta \sim \text{Dir}(\alpha)$$

```python
if node is self.root and self.root_mix > 0:
    noise = np.random.dirichlet([DIRICHLET_ALPHA] * len(choices))
    policy = policy.clone()
    policy[choices] = policy[choices] * (1 - self.root_mix) + torch.from_numpy(noise).float() * self.root_mix
```

El parámetro $\alpha$ controla cuánto se concentra. Con $\alpha$ grande el sorteo sale casi plano y vuelves a tener la mezcla uniforme con pasos intermedios para disimular. Con $\alpha$ pequeño sale muy desigual. AlphaZero usa `0.03` en Go, que tiene `361` casillas, y la regla habitual es escalarlo con el número de jugadas legales, cosa que en un `5x5` deja algo cercano a `1.0`.

**La misma cantidad de ruido, distinta forma.** Medido sobre la posición de apertura, la búsqueda pasa de mirar dos casillas a mirar catorce.

Hay un segundo beneficio que yo llevaba meses sin recibir, que es que al ser distinto en cada partida, dos partidas de self-play desde la misma posición ya no exploran lo mismo. La mezcla plana es determinista, con lo cual mis cincuenta partidas por iteración recibían exactamente el mismo empujón en las mismas direcciones. 50 empujones idénticos son un empujón, en definitiva.

## Arreglo 2

`C_PARAM = 1.0`.

En PUCT he visto que la gente en algunas repos suelen andar entre `1` y `2.5`, y yo lo había dejado en el extremo bajo del rango sin ningún motivo particular, básicamente porque un día lo bajé y me fui al gym. Subirlo a `2.5` divide el techo por más de seis.

Es la constante con mejor relación esfuerzo-resultado de todo el proyecto y llevaba ahí desde la primera versión del árbol, con un comentario mío al lado en [[HEX-08 Cuánta intuición hace falta para ser experto]] explicando que UCB1 y PUCT no comparten magnitudes. O sea que escribí el diagnóstico correcto en su momento y luego no apliqué el tratamiento, que es exactamente lo que hace la gente con el informe médico que le dan en la revisión anual.

Lo comprobé sin reentrenar nada, corriendo la suite de aperturas con la red vieja y cambiando solo esa constante desde fuera del módulo. `15` de `25` con `1.0`. `17` de `25` con `2.5`.

Dos aperturas más, gratis, con una red ya entrenada y sin tocarle un solo peso.

## Arreglo 3

Los dos anteriores hacen que la búsqueda mire más sitios. Ninguno garantiza que el conjunto de datos cubra las posiciones que hacen falta.

Porque los dos lados de una partida de self-play son la misma red, con los mismos gustos. Una línea que ambos consideran mala no aparece jamás, y no precisamente porque la exploración falle, sino porque para llegar a esa posición harían falta varias jugadas seguidas que ninguno de los dos va a hacer ni por accidente.

Con lo cual, la solución más tonta posible.

```python
if np.random.rand() < 0.5:
    for _ in range(np.random.randint(1, 3)):
        Engine.place(*Engine.index_to_cell(int(np.random.choice(Engine.valid_choices()))))
```

Arrancas la mitad de las partidas colocando una o dos fichas **al azar** antes de que la red juegue nada. Y ya está. Cuatro líneas y una de ellas es un `if`.

No depende de la policy, ni del value, ni de PUCT, ni de ninguna constante calibrada contra nada. Fuerza cobertura por construcción y no hay nada dentro del sistema que pueda anularla.

Después de dos días persiguiendo un umbral analítico, con logaritmos y raíces y una discusión conmigo mismo sobre desigualdades de concentración, que la solución robusta sea "empieza en otro sitio" tiene su punto humillante y bastante de instructivo. La lección, que creo que se generaliza fuera de este proyecto, es que cuando un sistema tiene un punto ciego causado por su propia dinámica, la salida suele estar fuera de la dinámica. No dentro. Ajustar los parámetros del bucle no te saca del bucle.

## Dos cosas más que iban en el mismo lote

**El replay buffer**, contado ya en [[HEX-09 Self-play]], que consiste sencillamente en no tirar las partidas de cada iteración. Diez tandas en vez de una.

```python
buffer = deque(maxlen=BUFFER_ITERATIONS)
buffer.append(fresh)
all_examples = [ex for piece in buffer for ex in piece]
```

**La simetría de `180` grados**, que duplica el conjunto de datos multiplicando el coste por cero, que es el mejor negocio del cuaderno.

Esas dos las metí en un entrenamiento anterior, ellas solas, sin nada más. Y el resultado fue una mejora modesta. Modesta de verdad, no modesta de las que uno dice para no presumir. Contra el bot externo, `40` de `50` antes y `41` de `50` después, que en cincuenta partidas es ruido con esmoquin.

Lo cual es un dato, no un fracaso, y es un dato que me habría ahorrado tiempo si lo hubiera tenido antes. El buffer y la simetría atacan el olvido y el tamaño del conjunto. No atacan la exploración.

Y mi problema era de exploración. Estuve arreglando la fontanería de un piso donde el problema era que no había agua en la calle.

## El diagnóstico que dejé corriendo

Antes de lanzar el entrenamiento largo metí dos números en el registro de cada iteración, porque ya sabía perfectamente que la pérdida no me iba a decir la verdad y no me apetecía volver a pasar por eso.

Las **jugadas efectivas**, que son $e^{H(p)}$ sobre un conjunto fijo de doscientas posiciones. Y el **valor absoluto mediano del value** sobre esas mismas posiciones.

Detalle importante que casi se me pasa. Las posiciones tienen que ser **siempre las mismas**. Si en cada iteración sorteas posiciones nuevas, la métrica se mueve por el sorteo y no por la red, y no puedes comparar la iteración `10` con la `90`. Se generan una vez con semilla fija y se reutilizan hasta el final.

Y las genero jugando al azar, con lo cual muchas son posiciones que ninguna red decente visitaría jamás. Eso está hecho a propósito, porque lo que quiero medir es precisamente si la red tiene opinión fuera de sus líneas favoritas. Medir a alguien solo en su terreno es lo que llevaba haciendo cuatro meses.

## Cómo fue

Con la configuración vieja, las jugadas efectivas caen a `3.28` en la iteración `24` y se quedan clavadas ahí durante setenta y cinco iteraciones más. El value sube monótonamente hasta `0.998`. A veces más y a veces menos, pero llega ahí.

Con los arreglos, se estabilizan alrededor de `6` sobre la iteración `24` y luego **suben**, terminando en `7.58`. El value oscila entre `0.85` y `0.97` durante las últimas sesenta iteraciones sin escaparse hacia arriba.

Y hay un detalle en esa segunda tabla que me gustó especialmente. En la iteración `71` el value toca su mínimo local de `0.8495` y las jugadas efectivas tocan su máximo de `7.72`.

En la misma fila. Las dos.

No es casualidad. Menos certeza deja más hueco para explorar, que es exactamente el mecanismo que llevo tres notas persiguiendo, funcionando por fin en la dirección que me conviene.

## El mapa de aperturas

Con la configuración vieja, desde la iteración `5` la política de apertura reparte el `100%` de las visitas entre dos casillas, al `50%` cada una, y no se mueve de ahí en noventa y cinco iteraciones. Un motor con dos aperturas en su repertorio. Dos.

Con la nueva, el centro se lleva entre el `78%` y el `92%`, y hay otras diez u once casillas con visitas distintas de cero. Eso sí es un repertorio, con una apertura preferida y alternativas que sigue considerando por si acaso.

Por cierto, las dos casillas del caso viejo eran `(1,3)` y `(3,1)`, que son imagen especular una de la otra bajo la rotación de `180` grados, y se repartían el `50.7%` y el `49.3%`. O sea que mi aumentación por simetría estaba funcionando con una precisión escalofriante sobre un motor que por lo demás estaba completamente roto. Una pieza impecable dentro de una máquina inútil.

Que es, si lo pienso, una imagen bastante fiel de cómo ha ido este proyecto entero.

Seguimos con [[HEX-15 Un metro que no se dilata]].

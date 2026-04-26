Cross-entropy es la función de coste que usé en lugar del clásico error cuadrático. La pregunta natural cuando te dicen "vamos a usar otra función de coste" es por qué. Si el error cuadrático funciona en mil sitios, ¿por qué cambiamos justo aquí?

La respuesta tiene que ver con que la salida de la red es una distribución de probabilidad (gracias a [[NN-04 Softmax|softmax]]), no un valor numérico cualquiera. Y para distribuciones de probabilidad hay funciones de coste que se llevan mejor con la geometría del problema que el error cuadrático.

## La fórmula

$$C = -\log(\hat{y}_{\text{correcto}})$$

Donde $\hat{y}_{\text{correcto}}$ es la probabilidad que la red le asignó al dígito correcto. Y ya está. Eso es cross-entropy en su forma más simple para clasificación.

Sí, parece increíblemente sencillo. Y lo es. Si la red predijo 0.85 para el dígito correcto, el coste es $-\log(0.85) \approx 0.16$, un número pequeño porque la red lo está haciendo bastante bien. Si la red predijo 0.001 para el dígito correcto, el coste es $-\log(0.001) \approx 6.9$, un número grande porque la red está fallando estrepitosamente.

## Por qué el logaritmo

Aquí está la gracia. La función $-\log(x)$ tiene una propiedad muy concreta: cuando $x$ se acerca a 0, $-\log(x)$ se va a $+\infty$. Cuando $x$ se acerca a 1, $-\log(x)$ se acerca a 0. Eso significa que cross-entropy **penaliza muchísimo más a las predicciones muy malas**. Y eso es lo que queremos.

Compara con el error cuadrático. Si la red predice 0.001 cuando la respuesta correcta es 1, el error cuadrático es $(1 - 0.001)^2 \approx 0.998$. Un número grande, sí, pero acotado. Por mucho que se equivoque la red, el error cuadrático no pasa de 1. Cross-entropy, en cambio, da casi 7 en el mismo caso. Y si la red diera 0.0001 daría casi 9.2. Y si fuera $10^{-10}$, daría 23. La penalización es brutal cuando la red mete la pata fuerte.

Eso es bueno porque hace que gradient descent empuje los pesos con muchísima más fuerza cuando la red está completamente perdida. La red aprende mucho más rápido en los casos donde más falta hace.

## Por qué solo el dígito correcto

Esta parte tiene su miga. La definición general de cross-entropy es:

$$C = -\sum_{i} y_i \cdot \log(\hat{y}_i)$$

Donde $y$ es el vector one-hot de la respuesta correcta y $\hat{y}$ es la predicción. Pero en clasificación con [[CON-one-hot encoding|one-hot]], el vector $y$ tiene un solo 1 (en la posición del dígito correcto) y ceros en las demás. Entonces todos los términos del sigma donde $y_i = 0$ se anulan. Solo sobrevive el término donde $y_i = 1$, que es exactamente el del dígito correcto. Y ese término se reduce a $-\log(\hat{y}_{\text{correcto}})$.

Es decir, la fórmula completa de cross-entropy en general suma sobre todas las clases, pero en clasificación de una sola etiqueta correcta se simplifica a "menos el logaritmo de la probabilidad asignada a la respuesta correcta". Por eso $C = -\log(\hat{y}_{\text{correcto}})$ es la versión que está en el código como tal.

## Cross-entropy + softmax = DIOS

Como mencioné en la nota de softmax, la combinación de estas dos funciones tiene una propiedad mágica: cuando derivas el coste respecto a la pre-activación de la última capa, todo se simplifica a:

$$\frac{\partial C}{\partial z} = S - y$$

Predicción menos realidad. Esa es la dirección del error en la última capa, y es el punto de partida de [[NN-07 Backpropagation|backpropagation]]. No tienes que derivar softmax con sus dos casos por separado, no tienes que derivar el logaritmo de cross-entropy y multiplicar por todo. Las dos funciones se cancelan algebraicamente y queda esa resta limpia.

Esto no es coincidencia. Softmax y cross-entropy se diseñaron pensando una en la otra. Si usaras error cuadrático con softmax, perderías esta cancelación y backprop sería un dolor de cabeza. Si usaras cross-entropy con sigmoid en la salida, también. La pareja entera tiene sentido junta.

## La implementación

En código, cross-entropy con un solo ejemplo es así:

```python
def cross_entropy(predictions, correct_index):
    loss = -np.log(predictions[correct_index])
    return loss
```

Cuando llega [[NN-10 Mini-batches|mini-batches]] esta función cambia para procesar un batch entero, y se mete un truquillo más para evitar problemas numéricos cuando una predicción es exactamente cero (porque $\log(0) = -\infty$). Pero eso lo cuento allí.

## Recapitulación

Cross-entropy es la función de coste estándar para clasificación con softmax.

Su forma en este proyecto es $C = -\log(\hat{y}_{\text{correcto}})$, el menos logaritmo de la probabilidad que la red asignó a la respuesta correcta.

Penaliza desproporcionadamente las predicciones muy malas, lo cual hace que la red aprenda rápido cuando se equivoca mucho.

Combinada con softmax, su derivada se simplifica a $S - y$, que es el regalo más grande que te puede hacer una función de coste.

Con esto cerrado, ya tenemos todas las piezas matemáticas: sigmoid, softmax, cross-entropy, sus derivadas. Toca ya construir la arquitectura: las clases `Layer` y `NeuralNetwork`. Eso es [[NN-06 La arquitectura — Layer y NeuralNetwork]].
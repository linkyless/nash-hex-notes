Después de tener las funciones matemáticas guays (sigmoid, softmax, cross-entropy y sus derivadas), tocaba meterlas dentro de una estructura. Una red neuronal es un **objeto** que tiene capas, y cada **capa** tiene pesos, biases, y sabe cómo procesar entradas. Es decir, toca usar POO (programación orientada a objetos).

La parte de POO en Python no la dominaba bien al empezar el proyecto. Sabía qué era una clase conceptualmente, pero implementar una con sus métodos y atributos era otra cosa. Es más, sabía cómo hacerlo en Python, pero más o menos se me olvidó un poco. Así que esta nota tiene también un trozo de "aprender Python" mezclado con "diseñar una red neuronal".

## La clase Layer

Una capa de la red tiene tres cosas: una matriz de pesos, un vector de biases, y una función de activación. Cuando creo una capa tengo que decirle cuántas neuronas tiene la capa anterior y cuántas tiene esta capa. Con eso puedo dimensionar la matriz de pesos correctamente.

```python
class Layer:
    def __init__(self, n_prev, n_curr, activation):
        self.weights = np.random.randn(n_curr, n_prev) * 0.01
        self.bias = np.zeros(n_curr)
        self.activation = activation
```

Hay varias cosas aquí que vale la pena comentar.

**La forma de la matriz de pesos.** Es `(n_curr, n_prev)`, es decir, filas por neuronas de la capa actual y columnas por neuronas de la capa anterior. Si la capa anterior tiene 784 neuronas y esta tiene 256, la matriz es de $256 \times 784 = 200{.}704$ pesos. Cada fila $j$ contiene los 784 pesos que conectan a la neurona $j$ de esta capa con las 784 de la anterior. Esa convención hace que la fórmula $z = Wa + b$ funcione directamente con multiplicación de matrices: si $a$ es un vector de 784 elementos, $Wa$ da un vector de 256.

**Inicialización aleatoria de los pesos.** Si inicializas los pesos a cero, todas las neuronas de la capa aprenden exactamente lo mismo durante el entrenamiento porque empiezan idénticas y reciben los mismos gradientes. Eso se llama "problema de simetría" y es uno de los errores típicos al implementar esto por primera vez. La solución es inicializar con números aleatorios, así cada neurona empieza siendo diferente y desarrolla su propia "personalidad" durante el entrenamiento.

**El factor `* 0.01`.** Esa multiplicación no es un detalle estético, es importante. Si los pesos son demasiado grandes, las pre-activaciones se descontrolan y sigmoid se satura (se queda pegada al 0 o al 1, donde su derivada es casi nula y la red no aprende). Si son demasiado pequeños, las activaciones se desvanecen y la red tampoco aprende. Multiplicar por 0.01 los mantiene en una escala razonable. Hay inicializaciones más sofisticadas (Xavier, He) que afinan ese factor según el tamaño de la capa anterior, y de hecho hago el cambio a He más adelante junto con otras mejoras. Pero por ahora, 0.01 va bien.

**Los biases a cero.** Los biases sí que pueden empezar a cero sin problemas, porque no tienen el problema de simetría: cada uno está asociado a una neurona única, así que aunque empiecen iguales, los gradientes serán distintos para cada uno desde la primera iteración.

**La función de activación pasada como parámetro.** Esta fue una de mis decisiones favoritas del proyecto. En lugar de hardcodear sigmoid dentro de la clase, paso la función como argumento. Eso me permite crear capas con activaciones diferentes simplemente pasándole `sigmoid` o `softmax` al constructor. En Python las funciones son objetos como cualquier otro y se pueden pasar como parámetros. Eso se llama "first-class functions" y es algo que conocía vagamente pero que aquí me hizo el código mucho más limpio:

```python
hidden_layer = Layer(784, 256, sigmoid)
output_layer = Layer(256, 10, softmax)
```

Sin esa flexibilidad, entonces, tendría que tener un `if` dentro de `forward` para decidir qué aplicar dependiendo del tipo de capa, o tener dos clases distintas. Esto es elegante, a mí no me digas nada, yo lo veo elegante.

## El forward pass de una capa

El método `forward` de la capa hace exactamente lo que dice la fórmula central de las redes neuronales: $z = Wa + b$, después aplica la función de activación.

```python
def forward(self, activations):
    new_act = (self.weights @ activations) + self.bias
    self.Z = new_act
    self.S = self.activation(new_act)
    return self.S
```

El operador `@` es la multiplicación de matrices en numpy, que internamente usa BLAS (una librería en C súper optimizada). Si yo escribiera esa multiplicación con tres bucles for en Python sería decenas o cientos de veces más lenta. Aquí es donde se nota la diferencia entre escribir matemáticas a mano y escribir matemáticas con la herramienta correcta.

En esta versión, `activations` es un vector de 784 elementos (una sola imagen). La multiplicación $W \cdot a$ con $W$ de forma $(256, 784)$ y $a$ de forma $(784,)$ devuelve un vector de 256, que es la pre-activación de la capa actual. Más adelante, cuando llegue [[NN-10 Mini-batches|mini-batches]], digamos que esto cambia: en lugar de pasar una imagen procesamos 32 a la vez, y la fórmula se reescribe para que tenga sentido sobre matrices. Pero por ahora, una imagen, un vector.

Hay un detalle importante: guardamos `self.Z` (la pre-activación) y `self.S` (la activación). ¿Por qué? Porque cuando llegue el momento de [[NN-07 Backpropagation|backpropagation]] vamos a necesitar estos valores para calcular los gradientes. Sin guardarlos, tendríamos que recalcularlos durante el backprop, lo cual sería una mierda. Un poco de caching, vamos.
## La clase NeuralNetwork

Una vez que tengo `Layer`, una red neuronal es básicamente una lista de capas con un par de métodos para hacer forward pass y backpropagation. Lo bonito es que el forward pass de la red entera es ridículamente simple:

```python
class NeuralNetwork:
    def __init__(self, layers):
        self.layers = layers
    
    def forward(self, data):
        for layer in self.layers:
            data = layer.forward(data)
        return data
```

Sí, eso es todo. Recorres las capas en orden, en cada paso pasas la salida de la capa anterior como entrada de la siguiente, y al final devuelves lo que sale de la última capa. Tres líneas. Cuando lo escribí no me lo creía. Después de toda la teoría sobre redes neuronales, propagación, capas, activaciones, el forward pass se reduce a un bucle for. Esa simplicidad es la belleza de las matemáticas como bien planteadas o algo así. Toda la complejidad está en los pesos y en los datos, no en la estructura del código.

El método `backprop` es muchísimo más complicado y tiene su propia nota porque se merece atención completa. Pero por ahora, la red ya sabe predecir (con pesos aleatorios, mal, pero predice). El siguiente paso es enseñarle a aprender.

## Probarlo antes de seguir

Antes de avanzar, escribí una prueba rápida para asegurarme de que el forward pass funciona:

```python
pred = np.random.randn(784)
A = Layer(784, 128, sigmoid)
Exit = Layer(128, 10, softmax)
network = NeuralNetwork([A, Exit])
print(network.forward(pred))
print(np.sum(network.forward(pred)))
```

Le metes un vector aleatorio de 784 valores (simulando una imagen), pasas por una red con capa oculta de 128 y salida de 10, y miras qué sale. Si softmax está bien implementada, las 10 salidas deben sumar 1. Y suman algo como 0.9999999999999998, que es 1 con error de coma flotante. Pero vamos, que funciona.

Ese momento, la primera vez que vi una red neuronal mía haciendo un forward pass válido, fue raro. Sabía que no estaba prediciendo nada útil porque los pesos eran aleatorios. Pero matemáticamente, lo que acababa de pasar es un forward pass real de una red neuronal de verdad. La estructura ya estaba.

## Recapitulación

`Layer` guarda pesos, bias, y función de activación. Su `forward` calcula $z = Wa + b$, aplica la activación, y guarda los valores intermedios para backprop.

`NeuralNetwork` es una lista de capas. Su `forward` recorre las capas y encadena las salidas con las entradas. Tres líneas.

Decisiones de diseño importantes: inicialización aleatoria de pesos para evitar simetría, biases a cero, función de activación como parámetro para flexibilidad, y caching de valores intermedios para backprop.

La red ya predice, aunque mal. Toca enseñarle a aprender, y eso es [[NN-07 Backpropagation|backpropagation]].
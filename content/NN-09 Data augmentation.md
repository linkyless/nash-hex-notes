Como conté en [[NN-08 Implementación con MNIST]], la red llegaba al 97.82% sobre el conjunto de test pero la realidad en el visualizador web era otra. Los dígitos escritos con el ratón fallaban con bastante frecuencia, sobre todo cuando no estaban perfectamente centrados o cuando el trazo era distinto al estilo MNIST.

La razón es que la red había aprendido a reconocer dígitos en unas condiciones muy específicas (centrados, mismo tamaño, mismo estilo de trazo) y cualquier desviación la confundía. La red no era robusta. Era buena en su mundo y mala fuera de él.

## La idea era enseñarle más variedad, manteniendo la misma base de datos

Data augmentation consiste en generar versiones modificadas de las imágenes durante el entrenamiento. Mismas imágenes, pero rotadas, desplazadas o escaladas aleatoriamente. La idea es que la red vea muchas variaciones de cada dígito y aprenda que un siete sigue siendo un siete aunque esté un poco a la izquierda o ligeramente inclinado.

Hay dos enfoques:

**Generar todas las variaciones por adelantado** y guardarlas en un dataset más grande. De 60.000 imágenes pasarías a 300.000. Más memoria, pero más simple.

**Generar variaciones aleatorias durante el entrenamiento**. Cada vez que la red ve una imagen, recibe una versión transformada al vuelo. Cero memoria extra, y cada época la red ve versiones distintas. Esto se llama "augmentation on the fly" y es lo que hice.

## Las transformaciones

Para no complicarme, apliqué solo dos transformaciones:

**Rotación** entre $-15°$ y $+15°$. Suficiente para que la red aprenda que un siete inclinado sigue siendo un siete.

**Desplazamiento** entre $-3$ y $+3$ píxeles en cada eje. Suficiente para que aprenda que un dígito un poco descentrado sigue siendo el mismo dígito.

Para hacer estas transformaciones usé `scipy.ndimage`, que tiene funciones específicas:

```python
from scipy.ndimage import shift, rotate
```

`rotate(image, angle, reshape=False)` rota la imagen un ángulo dado. El `reshape=False` mantiene el tamaño original (sin él, la imagen rotada se haría más grande para acomodar las esquinas).

`shift(image, [dy, dx])` desplaza la imagen `dy` píxeles vertical y `dx` horizontal.

## El cambio en el bucle de entrenamiento

Para que augmentation funcione, las transformaciones tienen que aplicarse a la imagen $28 \times 28$, no al vector aplanado de 784. Eso obligó a modificar cómo cargo los datos: dejé el train sin aplanar y aplano dentro del bucle, justo después de aplicar las transformaciones.

```python
x_train = x_train / 255.0  # solo normalizar, no aplanar
```

Y el bucle:

```python
for epoch in range(epochs):
    for i in range(len(x_train)):
        image = x_train[i]
        
        angle = np.random.uniform(-15, 15)
        dx = np.random.randint(-3, 4)
        dy = np.random.randint(-3, 4)
        image = rotate(image, angle, reshape=False)
        image = shift(image, [dy, dx])
        
        data = image.reshape(784)
        predictions = network.forward(data)
        network.backprop(data, predictions, y_train[i], learning_rate)
```

`np.random.uniform(-15, 15)` da un decimal entre $-15$ y $15$. `np.random.randint(-3, 4)` da un entero entre $-3$ y $3$ (el límite superior es exclusivo en numpy). Cada imagen recibe una transformación aleatoria distinta cada vez que la ve, así que en 10 épocas la red ve efectivamente 10 variaciones de cada imagen.

## El efecto extraño

Lo curioso de data augmentation es que la precisión sobre el conjunto de test bajó ligeramente, de 97.82% a 96.49%. Eso parece peor a primera vista. Pero hay un matiz importante: el conjunto de test también tiene los dígitos perfectamente centrados, igual que el conjunto de entrenamiento original. Como la red ahora ya no se sobreajusta tanto a esa estética concreta, su precisión "nominal" baja. ¿Se entiende?
Pero —bien hecho— su rendimiento real sobre dígitos dibujados a mano subió bastante. La red ahora reconoce mucho mejor mis sietes con cruz, mis cuatros descentrados, mis ochos un poco torcidos. La métrica del test set no captaba esa robustez, pero estaba ahí.
Las métricas oficiales en ML, supongo, no siempre reflejan lo que importa en la práctica. Una red puede tener mejor accuracy sobre un benchmark y ser peor en el mundo real, o viceversa. *La métrica es una proxy*, decía mi tatarabuelo.

## Recapitulación

Data augmentation genera variaciones aleatorias de las imágenes durante el entrenamiento (rotación, desplazamiento) para que la red aprenda a reconocer dígitos en condiciones más variadas.

Las transformaciones se aplican sobre la imagen $28 \times 28$ antes de aplanar y meter en la red.

La precisión sobre el conjunto de test bajó un poco, pero el rendimiento sobre dígitos reales mejoró bastante. Las métricas oficiales no siempre reflejan lo que importa.

La siguiente mejora fue la más drástica del proyecto: pasar de procesar imágenes una a una a procesarlas en grupos de 32 con [[NN-10 Mini-batches]].
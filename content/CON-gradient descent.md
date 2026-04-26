Gradient descent o el Descenso del Gradiente es el algoritmo que ajusta los parámetros de una red para reducir el coste. Si la derivada del coste respecto a un parámetro te dice en qué dirección crece el coste, te mueves en la dirección opuesta.

La regla de actualización para un peso es:

$$w \leftarrow w - \eta \cdot \frac{\partial C}{\partial w}$$

Donde $\eta$ es el learning rate, un número pequeño que controla cuánto te mueves en cada paso. Si $\eta$ es muy grande, das saltos enormes y puedes pasarte el mínimo. Si es muy pequeño, tardas demasiado en llegar.

La intuición geométrica es la de bajar una montaña con niebla. No ves dónde está el valle, pero sí sientes la pendiente bajo tus pies. La derivada te dice la pendiente. Si das pasos pequeños siguiendo la pendiente hacia abajo, eventualmente llegas a un mínimo.

El problema es que el "paisaje" del coste en una red neuronal tiene cientos de miles de dimensiones (una por cada peso). No puedes visualizarlo, pero la idea matemática es la misma: el gradiente es la dirección de máxima pendiente, y avanzar en sentido contrario reduce el coste.

[[NN-07 Backpropagation|Backpropagation]] es lo que calcula los gradientes para todos los pesos a la vez. Gradient descent es lo que los aplica.

Hay variantes, por ejemplo la SGD (stochastic gradient descent, actualiza con una sola muestra), mini-batch (actualiza con un grupo, ver [[NN-10 Mini-batches]]), y otros más sofisticados como Adam que ajustan el learning rate dinámicamente. En este proyecto uso mini-batch, y usé inicialmente SGD.
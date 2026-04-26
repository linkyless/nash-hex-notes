
Softmax apareció en escena cuando ya creía que tenía la red controlada. Tenía sigmoid funcionando, había escrito las funciones básicas, había pensado un par de minutos sobre la arquitectura. Y entonces, casi de pasada, vi un vídeo que comentaba que "para la capa de salida no se usa sigmoid, sino que se usa softmax". Y ahí pensé yo: ¿perdona? ¿Eh?¿Por qué cambiamos de función justo en la última capa? ¿Qué tiene de malo sigmoid? Si funciona en la capa oculta, ¿por qué no en la salida?

## ¿Qué problema resuelve softmax?

Imagina que la capa de salida tiene 10 neuronas con sigmoid. Cada neurona tiene una activación entre 0 y 1, perfecto. Pero esas activaciones son **independientes**. Cada neurona aplasta su propio número sin enterarse de lo que están haciendo las demás. Eso significa que puedes acabar con una salida tipo $[0.8, 0.7, 0.9, 0.6, 0.85, ...]$, donde varias neuronas tienen valores altos al mismo tiempo. Y eso no tiene sentido como predicción de un solo dígito. Si la red está prediciendo "qué dígito es esto", lo que quieres es que las 10 neuronas representen una distribución de probabilidad, todas entre 0 y 1, pero **sumando exactamente 1**. Si una neurona dice 0.85, las otras nueve solo pueden sumar 0.15 entre todas. Esa es la limitación que sigmoid no impone, y softmax sí.

Softmax convierte un vector cualquiera de números (positivos, negativos, lo que sea) en una distribución de probabilidad válida sobre las 10 clases. Cada salida está entre 0 y 1 y todas suman 1. La neurona con el número más alto se lleva la mayor probabilidad, las demás se reparten el resto.

## La fórmula

$$\text{softmax}(x_i) = \frac{e^{x_i}}{\sum_{j} e^{x_j}}$$

Donde $x_i$ es el valor de la neurona $i$ antes de softmax (es decir, la pre-activación $z_i$ que sale de $Wa + b$), y la suma del denominador recorre todas las neuronas de la capa.

A primera vista la fórmula es rara. Si solo quieres "convertir números en probabilidades que sumen 1", lo más obvio sería dividir cada número entre la suma total. Algo como $\frac{x_i}{\sum_j x_j}$. Pero eso tiene un problema enorme: ¿qué pasa si alguno de los $x_i$ es negativo? Tendrías probabilidades negativas, y eso es matemáticamente absurdo y, por supuesto, físicamente imposible.

Por eso softmax pasa cada número por la exponencial primero. La exponencial $e^x$ siempre devuelve algo positivo. Si $x = 2$, $e^x \approx 7.4$. Si $x = -3$, $e^x \approx 0.05$. Si $x = 0$, $e^x = 1$. Da igual el signo de la entrada, la salida siempre es positiva. Una vez que tienes todos los números positivos, ya puedes dividir entre la suma total y tienes tu distribución.

## Lo que softmax tiene de astuto

Softmax exagera las diferencias. Si un valor es solo un poco más grande que los demás, después de aplicar $e^x$ esa diferencia se amplifica bastante, porque la exponencial crece muy rápido. Eso hace que la red tienda a dar predicciones muy seguras, en lugar de repartir la probabilidad de forma uniforme.

Ejemplo concreto: si la pre-activación es $[1, 2, 3]$, las exponenciales son aproximadamente $[2.7, 7.4, 20.1]$. La suma es alrededor de 30.2. Las probabilidades finales son $[0.09, 0.24, 0.67]$. La diferencia entre el 1 y el 3 era pequeña, pero softmax la convirtió en una predicción del 67% para el último valor. Eso es bueno porque queremos que la red sea decisiva, no que reparta probabilidades de forma cobarde. Eso está guay.

## Por qué softmax solo va al final

Esto me costó verlo claro al principio. ¿Por qué no usar softmax en todas las capas? La respuesta es que solo tiene sentido cuando lo que quieres es una distribución de probabilidad sobre clases mutuamente excluyentes. En la capa oculta, las neuronas no representan clases, representan "características intermedias" que la red está descubriendo. No tiene sentido forzar que sumen 1, eso destruiría la información. Sigmoid en la capa oculta deja que cada neurona se active independientemente, lo cual es lo que queremos.

Softmax solo aparece en la última capa porque es la única donde tienes una decisión categórica final. "¿Es un cero, un uno, un dos...?". Y solo puede ser una de esas cosas. Por eso suma 1. Es mucho más intuitivo.

## La derivada de softmax (casi rompo la pizarra con esta mierda)

Después de derivar sigmoid limpiamente, esperaba que softmax fuera parecido. Ja, que gracioso eres. Adelante, deriva un puto SUMATORIO que tenga OTRAS MIERDAS ENCIMA. Softmax tiene esta particularidad que la hace mucho más complicada de derivar.

Mira la fórmula otra vez:

$$\text{softmax}(x_i) = \frac{e^{x_i}}{\sum_{j} e^{x_j}}$$

El denominador depende de **todos** los $x_j$, no solo de $x_i$. Eso significa que si cambias cualquier $x_j$, el resultado de $\text{softmax}(x_i)$ cambia, aunque $j$ no sea $i$. La función no es independiente neurona por neurona como sigmoid.

Eso te obliga a derivar dos casos distintos cuando aplicas la regla de la cadena. Cuando $j = i$ (estás derivando $\text{softmax}(x_i)$ respecto a su propio $x_i$) la cosa va de una forma. Cuando $j \neq i$ (estás derivando $\text{softmax}(x_i)$ respecto a otro $x_j$ que también vive en el denominador) la cosa va de otra. Es decir, la derivada de softmax no es un escalar simple por cada neurona, es una **matriz** entera donde cada celda dice cuánto afecta cada $x_j$ a cada $\text{softmax}(x_i)$. Qué gracioso somos todos.

Intenté hacer que la IA me lo explicara, pero me bloqueé, lo siento. Estaba con la cabeza intentando seguir la regla de la cadena con varios casos, derivadas parciales por todos lados, y no llegaba a un sitio limpio. Honestamente fue uno de los momentos más perdidos del proyecto.

## softmax + cross-entropy

Aquí viene lo bueno. Resulta que softmax se usa casi siempre junto con la función de coste cross-entropy ([[NN-05 Cross-entropy]]). Y cuando derivas la composición de softmax y cross-entropy juntas, TODA LA DERIVADA de softmax **se cancela**. El resultado final, increíblemente, es esto:

$$\frac{\partial C}{\partial z} = S - y$$

Donde $S$ es el vector de probabilidades que devolvió softmax y $y$ es el vector one-hot de la respuesta correcta. La derivada del coste respecto a la pre-activación de la última capa es simplemente "predicción menos realidad". Una resta de vectores. Eso es todo.

Cuando vi esto, pasé de la frustración total al alivio en menos de un minuto. No tienes que sufrir la derivada de softmax como matriz. La combinación con cross-entropy hace que se simplifique a una operación trivial. Por eso softmax y cross-entropy se diseñaron para usarse juntas. Locurón extremo. Igual, demostrar esa cancelación requiere meterse en la derivada de softmax con sus dos casos, multiplicarla por la derivada de cross-entropy, y ver cómo todo se simplifica. Es un buen ejercicio si quieres aceptarlo de verdad y no como acto de fe. Pero para que el proyecto funcione, basta con saber que el resultado es $S - y$ y usarlo. Lo apunté como deuda matemática pendiente, para hacerlo despacio cuando tenga más contexto. Déjenme disfrutar en ese aspecto.

## La implementación

En código, softmax queda muy limpia gracias a numpy:

```python
def softmax(arr):
    exp_arr = np.exp(arr)
    exp_sum = np.sum(exp_arr, axis=-1, keepdims=True)
    return exp_arr / exp_sum
```

`np.exp(arr)` aplica $e^x$ a cada elemento del array, sin bucles. `np.sum(..., axis=-1, keepdims=True)` suma a lo largo de la última dimensión y mantiene la forma para que la división funcione tanto si le metes un vector como si le metes un batch de vectores. Esos dos detalles (`axis=-1` y `keepdims=True`) son importantes cuando llegamos a [[NN-10 Mini-batches|mini-batches]], donde de repente softmax recibe matrices en lugar de vectores. Lo importante es que la fórmula matemática y el código se parecen una barbaridad, porque numpy está pensado para operar sobre vectores enteros sin que tengas que pensar en bucles.

## Recapitulación

Softmax convierte un vector de números cualesquiera en una distribución de probabilidad válida.

Funciona aplicando $e^x$ a cada elemento (que garantiza positividad) y luego dividiendo entre la suma total (que garantiza que sumen 1).

Solo se usa en la capa de salida, porque solo ahí queremos una distribución sobre clases mutuamente excluyentes.

Su derivada es complicada porque cada salida depende de todas las entradas, pero **cuando se usa con cross-entropy, la derivada combinada se simplifica a $S - y$**, lo cual es lo más bonito que vi en todo el proyecto.

Con softmax y cross-entropy juntas, la red ya tiene la última pieza para poder calcular el error de predicción y empezar a aprender. La siguiente nota es exactamente sobre eso: [[NN-05 Cross-entropy]].
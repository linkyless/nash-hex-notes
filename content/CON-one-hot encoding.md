One-hot encoding es la forma de representar una categoría como vector. Si tienes 10 clases (los dígitos del 0 al 9) y la respuesta correcta es la clase 3, el one-hot es:

$$y = [0, 0, 0, 1, 0, 0, 0, 0, 0, 0]$$

Un vector de ceros con un solo 1 en la posición de la clase correcta. El nombre "one-hot" viene de electrónica: una sola línea está "caliente" (en uno) y las demás están en cero.

En el proyecto aparece en dos sitios. En [[NN-05 Cross-entropy|cross-entropy]] como vector de la respuesta correcta para comparar con las predicciones de la red. En [[NN-07 Backpropagation|backprop]] al construir el vector $y$ para calcular $dZ_2 = S - y$, donde $S$ son las predicciones y $y$ es el one-hot de la respuesta correcta.

¿Por qué se usa? Porque las clases son categóricas y mutuamente excluyentes. Una imagen es un siete o un uno, no las dos cosas. El one-hot codifica esa exclusividad de forma natural.

En numpy se construye con `np.zeros(10)` y luego asignas un 1 a la posición correcta.
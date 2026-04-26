El producto exterior es una operación que toma dos vectores y devuelve una matriz. Si tienes $a$ de tamaño $n$ y $b$ de tamaño $m$, su producto exterior es una matriz de $n \times m$ donde el elemento $(i, j)$ es $a_i \cdot b_j$. Cada elemento de un vector multiplicado con cada elemento del otro.

En numpy se hace con `np.outer(a, b)`.

Aparece en [[NN-07 Backpropagation|backpropagation]] cuando calculamos el gradiente de los pesos. Tienes un vector de errores en una capa ($dZ$) y un vector de activaciones de la capa anterior ($S$), y necesitas una matriz del mismo tamaño que los pesos. La única operación que convierte dos vectores en una matriz con esa estructura es el producto exterior.

Cada elemento de la matriz de gradientes corresponde a un peso concreto que conecta dos neuronas. Y el gradiente de ese peso es justo "el error de la neurona destino multiplicado por la activación de la neurona origen". Eso es $dZ[i] \cdot S[j]$, exactamente la celda $(i, j)$ del producto exterior.

Cuando llega [[NN-10 Mini-batches]], el producto exterior se reemplaza por una multiplicación de matrices, que automáticamente suma los productos exteriores de las 32 imágenes del batch. Pero la idea conceptual es la misma.
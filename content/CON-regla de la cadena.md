La regla de la cadena dice que si una función depende de otra, que a su vez depende de otra, la derivada de la función entera es el producto de las derivadas de cada paso. Si tienes $C = f(g(h(x)))$, su derivada respecto a $x$ es:

$$\frac{dC}{dx} = \frac{dC}{dg} \cdot \frac{dg}{dh} \cdot \frac{dh}{dx}$$

Cada fracción es una derivada de un paso del camino. Las multiplicas y obtienes la derivada total.

En redes neuronales esto es central. El coste depende de la salida de la última capa, que depende de la pre-activación de la última capa, que depende de los pesos de la última capa, y así hacia atrás hasta los pesos de la primera capa. Para saber cuánto afecta un peso de la primera capa al coste, aplicas la regla de la cadena hacia atrás desde el coste hasta ese peso. Eso es exactamente lo que hace [[NN-07 Backpropagation|backpropagation]].

La intuición que más me sirvió fue que cada flecha del forward pass es una derivada parcial cuando lo recorres al revés. Míralo. El forward pass es $x \to Z_1 \to S_1 \to Z_2 \to S_2 \to C$. El backpropagation va al revés y multiplica una derivada por cada flecha.
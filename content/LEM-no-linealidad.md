---
title: LEM-no linealidad
tags:
  - lema
---
Dos capas lineales encadenadas son

$$W_2(W_1 x) = (W_2 W_1)x$$

o sea una capa lineal. Encadenando veinte pasa lo mismo, y lo único que se consigue es tardar veinte veces más en calcular lo que ya calculaba una.

Hace falta algo no lineal en medio, y evidentemente todos entendemos por qué, así que pasemos a la fórmula difícil.

$$\max(0, x)$$

Eso es **ReLU** y ese es el concepto entero. Corta los negativos a cero y deja pasar los positivos tal cual. Con eso la composición deja de ser lineal y las capas de arriba pueden detectar cosas que las de abajo no podían.

## Por qué no la sigmoide

$$\sigma(x) = \frac{1}{1+e^{-x}}$$

Su derivada nunca pasa de `0.25`. Como el gradiente se va multiplicando capa a capa al volver hacia atrás, en una red de diez capas llega multiplicado por algo del orden de $0.25^{10}$, que es aproximadamente nada. Ese es el desvanecimiento del gradiente y es la razón por la que las redes profundas no se entrenaban bien antes.

ReLU tiene derivada exactamente `1` donde está activa, así que el gradiente la atraviesa sin perder nada.

## El defecto simétrico

Una neurona ReLU cuya entrada sea siempre negativa tiene gradiente cero para siempre y deja de aprender. Se le llama neurona muerta.

Se arregla con variantes que dejan pasar un poco de los negativos, tipo `max(0.01x, x)`. Con tres capas ninguno de los dos problemas llega a manifestarse, así que la elección aquí es prácticamente convención.

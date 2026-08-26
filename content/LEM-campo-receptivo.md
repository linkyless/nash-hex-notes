---
title: LEM-campo receptivo
tags:
  - lema
---
Cuánto de la entrada original puede influir sobre una neurona concreta.

Se construye contando. Una neurona de la primera capa mira una ventana de `3x3` del tablero. Una de la segunda mira una ventana `3x3` de la salida de la primera, y cada casilla de esa ventana ya venía de mirar `3x3`, así que en total abarca `5x5`. Una de la tercera abarca `7x7`.

En general, para $L$ capas de kernel $k$,

$$R = 1 + L(k-1)$$

## Por qué importa aquí y no en otros sitios

Clasificando fotos, un gato ocupa una fracción de la imagen y da igual lo que haya en la esquina opuesta. Un campo receptivo modesto basta.

En Hex el objetivo es conectar dos bordes, y si una conexión puede depender de lo que pasa al otro lado del tablero, hace falta que alguna neurona vea el tablero entero.

Con tres capas de `3x3` sale $R = 7$, que en `5x5` cubre todo desde cualquier casilla y va sobrado. En `11x11` no llega, y despejando $L$ hacen falta cinco capas como mínimo.

Es de las pocas veces en que un hiperparámetro se justifica con una cuenta en vez de con "probé y funcionó".

## Un matiz

La influencia dentro del campo receptivo no es uniforme. Se concentra en el centro y se desvanece hacia los bordes, con lo cual el campo receptivo **efectivo** es bastante menor que el teórico.

O sea que la fórmula de arriba es una cota optimista y conviene margen. A esa profundidad, además, mantener el gradiente utilizable pide bloques residuales.

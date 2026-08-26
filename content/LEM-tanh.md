---
title: LEM-tanh
tags:
  - lema
---
$$\tanh(x) = \frac{e^x - e^{-x}}{e^x + e^{-x}}$$

Aplasta cualquier número real dentro del intervalo $(-1, 1)$, con el cero yendo al cero.

Va al final del value head, y hay dos razones independientes para ponerlo.

## Razón de escala

El MCTS usa el valor acumulado dividido entre visitas como término de explotación, y lo compara contra un término de exploración que vale del orden de uno o dos.

Una capa densa a secas puede escupir `6767`. Si la explotación puede valer eso, el término de exploración deja de existir en la práctica, no mueve la comparación, y la búsqueda se vuelve completamente codiciosa.

Con la salida acotada en $(-1,1)$, los dos términos son comparables por construcción.

## Razón de rango

El value se entrena contra resultados de partida, que en Hex son exactamente `+1` o `-1` porque no hay empates. Una red capaz de predecir `6767` está prediciendo algo que no ha ocurrido nunca ni va a ocurrir.

## Relación con la sigmoide

Son la misma familia, $\tanh(x) = 2\sigma(2x) - 1$. La diferencia es que tanh está centrada en cero, cosa que como activación intermedia importa y como activación de salida no, porque no hay más capas detrás.

## La pega

Con etiquetas que son exactamente `±1`, la pérdida cuadrática empuja la salida hacia los extremos del intervalo, donde la derivada de tanh tiende a cero. El resultado es un value cada vez más seguro de sí mismo y con menos gradiente para corregirse.

Medido en este proyecto, el valor absoluto mediano sube hasta `0.998` en el entrenamiento defectuoso y `0.949` en el corregido. Que un value seguro apague la exploración está contado en [[HEX-13 Doscientas veintiséis milésimas]].

Se puede suavizar entrenando contra `±0.95` en vez de `±1`, o bajando el peso del término del value. No lo he probado.

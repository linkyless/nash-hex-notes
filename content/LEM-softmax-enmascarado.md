---
title: LEM-softmax enmascarado
tags:
  - lema
---
$$p_i = \frac{e^{z_i}}{\sum_j e^{z_j}}$$

Exponencial para que todo salga positivo, división por la suma para que sume uno. Ya está, no hay más.

## El problema

A mitad de partida, la mitad de las casillas están ocupadas. Si el softmax reparte entre las veinticinco, parte de la probabilidad se va a sitios donde no se puede jugar, y esa es probabilidad que **no** se llevan las casillas legales.

## El truco

Se ponen los logits ilegales a menos infinito **antes** del softmax.

```python
valid = torch.zeros(TOTAL_CELLS, dtype=torch.bool)
valid[board.valid_choices()] = True
policy_logits[~valid] = -torch.inf
probs = torch.softmax(policy_logits, dim=0)
```

Como $e^{-\infty} = 0$, ese término desaparece del numerador y del denominador a la vez, y la normalización reparte toda la masa entre las legales sin que haya que hacer nada más.

Lo elegante es que no hace falta renormalizar después. Si se enmascarara al revés, primero softmax y luego poner ceros, quedaría un vector que ya no suma uno y habría que dividir otra vez, con el riesgo de dividir entre algo muy pequeño.

Falla si se enmascaran todas, porque entonces el denominador es cero y sale `NaN`. En Hex eso solo pasaría con el tablero lleno, que es terminal y donde la red no se consulta. Conviene tenerlo escrito en algún sitio de todas formas.

## El orden importa

La máscara usa índices del tablero real. La red, cuando le toca al segundo jugador, ha mirado el tablero traspuesto, con lo cual sus logits están en coordenadas traspuestas.

Enmascarar logits traspuestos con índices sin trasponer tapa las casillas equivocadas y produce un vector perfectamente normal, que suma uno y tiene ceros donde parece que debe tenerlos. Sin ninguna señal de error.

Primero se devuelven los logits al marco real, después se enmascara. El fallo correspondiente está en [[HEX-10 Algo anda mal]].

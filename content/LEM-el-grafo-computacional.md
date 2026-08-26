---
title: LEM-el grafo computacional
tags:
  - lema
---
Un tensor de PyTorch se acuerda de lo que le han hecho.

Cada operación queda registrada junto con el resultado y con quién fue el operando, y de ahí sale un grafo dirigido donde los nodos son operaciones y las aristas son tensores. Se construye solo mientras se escribe código que parece aritmética normal.

## Backward

`backward()` sobre un escalar recorre ese grafo hacia atrás hasta las hojas, aplicando la regla de la cadena en cada nodo y dejando en cada peso la derivada correspondiente.

O sea que el forward y el backward son el mismo recorrido en direcciones opuestas. El forward construye el camino y el backward vuelve por él repartiendo culpas. Por eso no se puede hacer backward sin un forward previo, porque no habría camino de vuelta.

Cuando un tensor alimenta a varias operaciones, sus contribuciones se suman. Eso es simplemente la regla de la cadena con varias variables.

## Acumulación

El campo `.grad` no se borra entre llamadas, se acumula. Sin `zero_grad()`, en el paso $n$ hay guardada la suma de los $n$ gradientes.

La dirección resultante no es disparatada, porque la suma de gradientes de varios lotes apunta más o menos donde apuntaría el gradiente del conjunto. Lo que se dispara es la magnitud, que crece linealmente con los pasos.

Ese mismo comportamiento es una técnica cuando se usa a propósito. Varios forward y backward sin poner a cero, un solo paso del optimizador, y tienes un lote grande simulado con memoria pequeña. La diferencia entre el bug y la técnica es dónde se coloca una línea.

## no_grad

`torch.no_grad()` apaga el registro. Dentro no se construye grafo, con lo cual las consultas a la red desde el árbol, donde no se entrena nada, salen mucho más baratas. Efecto secundario cómodo, esos tensores se convierten a numpy sin ceremonia.

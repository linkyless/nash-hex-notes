---
title: LEM-ruido de Dirichlet
tags:
  - lema
---
Se perturba el prior de la raíz del árbol antes de buscar, solo durante el self-play.

$$\tilde{P}(a) = (1-\lambda)P(a) + \lambda\,\eta_a$$

donde $\eta$ es un sorteo de una distribución de Dirichlet. En inferencia $\lambda = 0$ y no se toca nada.

## Qué sortea Dirichlet

Devuelve un vector de números positivos que suman uno, o sea una distribución sorteada al azar. El parámetro $\alpha$ controla cuánto se concentra.

Con $\alpha$ grande los sorteos salen casi planos y estás de vuelta en la mezcla uniforme con pasos intermedios para disimular. Con $\alpha$ pequeño salen muy desiguales, con casi toda la masa en una o dos componentes que cambian en cada sorteo.

## Por qué la mezcla plana no basta

La mezcla plana reparte $\lambda$ a partes iguales. Con $\lambda = 0.4$ sobre veinticinco casillas, a cada una le tocan `0.016`, y ese valor puede quedar por debajo del umbral que hace falta para recibir siquiera una visita. El cálculo del umbral está en [[HEX-13 Doscientas veintiséis milésimas]].

Dirichlet reparte esa misma masa de forma desigual, con lo cual dos o tres casillas por partida se llevan `0.05` o `0.10` y sí lo cruzan.

**La misma cantidad de ruido, distinta forma.** Medido aquí, la búsqueda pasa de mirar dos casillas en la raíz a mirar catorce.

Y hay un segundo beneficio que la mezcla plana no da. Al ser un sorteo, cada partida recibe un empujón distinto, así que dos partidas desde la misma posición divergen. La mezcla plana es determinista y las cincuenta partidas de una iteración recibían exactamente el mismo empujón, que es como no recibir ninguno.

## Cuánto vale alfa

AlphaZero usa `0.03` en Go con `361` casillas, `0.3` en ajedrez y `0.15` en shogi. La regla habitual es escalarlo con el número de jugadas legales, algo así como diez dividido entre el número de casillas.

Para un `5x5` eso deja alrededor de `0.4`. Aquí se usa `1.0` y el efecto medido ya es suficiente.

## Una pega honesta

El ruido cambia el reparto de visitas, y ese reparto es el objetivo de entrenamiento de la policy. Con lo cual se está entrenando en parte contra una búsqueda que has estropeado a propósito.

KataGo lo resuelve forzando visitas mínimas y luego restándolas al construir el objetivo, de modo que la búsqueda mira sin que el entrenamiento aprenda que hay que jugar ahí. Aquí no está implementado.

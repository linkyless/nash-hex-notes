---
title: LEM-temperatura
tags:
  - lema
---
El MCTS termina con un reparto de visitas y hay que convertirlo en una jugada. Coger el máximo es lo obvio y mata el entrenamiento.

Con la misma red y la misma posición, el máximo es siempre el mismo. Las cincuenta partidas de una iteración salen idénticas y acabas entrenando con veinticinco posiciones repetidas cincuenta veces.

La temperatura es el mando que gradúa entre coger el máximo y sortear.

$$\pi_\tau(a) = \frac{N(s,a)^{1/\tau}}{\sum_b N(s,b)^{1/\tau}}$$

Con $\tau = 1$ son las visitas normalizadas tal cual. Con $\tau$ tendiendo a cero el exponente se dispara, la más visitada aplasta a todas las demás, y en el límite recuperas el máximo. Con $\tau$ mayor que uno el exponente encoge las diferencias y todo se aplana.

El nombre viene de física estadística, de la distribución de Boltzmann, donde subir la temperatura hace que un sistema ocupe estados de más energía y por tanto se reparta más. La analogía es literal, la fórmula es la misma.

## Por qué no toda la partida

Porque al final las jugadas son forzadas. Si estás a un movimiento de ganar y el sorteo te manda a otro sitio, has tirado la partida, y encima has generado una etiqueta que dice que una posición ganada se pierde. Le estás enseñando a la red algo directamente falso.

Se sortea durante los primeros movimientos y luego se pasa al máximo. Aquí, `14` de veinticinco.

## Lo que no puede arreglar

La temperatura solo reparte entre lo que ya tiene visitas. Una casilla con **cero** visitas sigue teniendo probabilidad cero para cualquier $\tau$, porque cero elevado a lo que sea sigue siendo cero.

Comprobado subiendo hasta $\tau = 4$ sobre una posición donde veintitrés de las veinticinco casillas tenían cero visitas. De doscientos sorteos salieron dos aperturas distintas.

Con lo cual la temperatura es un amplificador de la diversidad que produzca la búsqueda, no una fuente de diversidad. Si la búsqueda mira dos casillas, la temperatura te da dos casillas. Amplificar el silencio no produce sonido.

Eso es lo que lleva a [[HEX-13 Doscientas veintiséis milésimas]], donde el problema resulta estar aguas arriba.

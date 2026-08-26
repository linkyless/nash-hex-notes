---
title: LEM-entropía
tags:
  - lema
---

$$H(p) = -\sum_i p_i \log p_i$$

Mide cuánta indecisión hay dentro de una distribución. Toda la masa en un sitio da cero. Repartida por igual entre $n$ opciones da $\log n$, que es el máximo.

Evidentemente todos entendemos qué significa esa fórmula, así que pasemos a una más difícil.

$$e^{H(p)}$$

Eso es la exponencial de la entropía y es el número que de verdad se usa aquí. Si la distribución es plana sobre veinte casillas devuelve `20`. Si está concentrada en una devuelve `1`. Si está a medias entre tres, algo cercano a `3`.

O sea que convierte una cantidad en nats, que no le dice nada a nadie, en un conteo de opciones que se lee sin traducir. Aquí se llaman **jugadas efectivas**. En ecología a esto mismo lo llaman número efectivo de especies y sirve para distinguir un bosque con diez árboles de diez tipos de otro con noventa robles y un pino.

## El suelo de la entropía cruzada

La policy se entrena comparando lo que dice la red contra lo que dijo la búsqueda,

$$H(\pi, p) = -\sum_i \pi_i \log p_i$$

y lo que hay que grabarse es que eso **nunca baja de $H(\pi)$**, con igualdad solo cuando las dos distribuciones coinciden.

Con una $\pi$ plana sobre veinticinco casillas, ese suelo es $\ln 25 \approx 3.22$. Durante las primeras iteraciones la pérdida es físicamente incapaz de bajar de ahí por muy bien que aprenda la red, y verla caer por debajo más tarde significa otra cosa distinta, que es que las etiquetas se han afilado.

## La consecuencia incómoda

Si el suelo depende del objetivo y no del modelo, entonces **la pérdida puede subir mientras el motor mejora**. Basta con que el objetivo se ensanche.

Medido. En el entrenamiento bueno la pérdida de policy toca fondo en la iteración `22` y sube durante las setenta y ocho restantes, mientras el rendimiento contra un rival externo pasa de `40/50` a `49/50`.

Con lo cual en este tipo de sistema la pérdida no mide calidad. Mide ajuste a un objetivo que se mueve, que es una cosa bastante distinta y que en aprendizaje supervisado normal no pasa, porque allí las etiquetas se quedan quietas.

La base del logaritmo da igual, por cierto. Cambiarla multiplica todo por una constante y las jugadas efectivas salen idénticas.

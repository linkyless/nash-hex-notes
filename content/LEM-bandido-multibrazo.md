---
title: LEM-bandido multibrazo
tags:
  - lema
---
Varias máquinas tragaperras, cada una con una probabilidad de pago distinta y desconocida. Un número limitado de monedas. ¿Cómo se reparten?

El conflicto es que cada moneda gastada en averiguar cuál es la buena es una moneda que no se gasta en la buena, y al revés. La única forma de obtener información es actuando.

Explotar siempre la mejor media observada falla, porque un error temprano se perpetúa y hay máquinas que nunca se vuelven a probar. Explorar por igual también falla, por motivos evidentes.

## UCB1

$$\frac{W_i}{N_i} + c\sqrt{\frac{\ln N}{N_i}}$$

El primer término es la media observada. El segundo es el margen de error alrededor de ella, que se estrecha conforme se acumulan muestras.

Sumados, dejan de compararse medias y pasan a compararse **los mejores valores que cada opción podría plausiblemente tener**.

De ahí sale lo que hace útil la fórmula. Una máquina puede ganar la comparación porque es buena o porque apenas se ha probado y la incertidumbre hace el trabajo. Explorar y explotar salen de una sola regla, sin ningún criterio explícito que decida cuándo hacer una cosa u otra.

El logaritmo ensancha lentamente el margen con el tiempo total, de modo que nada queda descartado para siempre. La raíz hace que el bono decaiga como $1/\sqrt{N_i}$.

El valor canónico $c = \sqrt{2}$ sale de la estimación del intervalo de confianza. No se hereda a PUCT, porque allí el término va multiplicado por un prior y las magnitudes cambian.

## Aplicación al árbol

Cada nodo se trata como una instancia del problema y sus hijos como máquinas. La diferencia es que aquí las recompensas cambian con el tiempo, dado que el valor de un hijo se mueve conforme su subárbol se desarrolla, así que la garantía teórica no se transfiere tal cual. Se usa igualmente porque funciona.

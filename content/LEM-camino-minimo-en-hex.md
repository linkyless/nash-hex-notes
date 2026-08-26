---
title: LEM-camino mínimo en Hex
tags:
  - lema
---
Ganar en Hex es conectar dos bordes, con lo cual la pregunta de cuánto te falta para ganar se puede medir exactamente en vez de estimarse.

Se construye un grafo sobre las veinticinco casillas y se le pone un coste a cada una según de quién sea.

- Fichas propias, `0`, porque ya son tuyas y atravesarlas es gratis.
- Casillas vacías, `1`, porque habría que gastar un turno en poner una ficha ahí.
- Fichas del rival, infinito, porque por ahí no se pasa.

El camino más barato de un borde tuyo al otro es entonces el número exacto de fichas que te faltan para ganar si el rival se quedara mirando. Dijkstra lo da en microsegundos.

## El bot

En tu turno se prueba cada casilla vacía, se coloca tu ficha ahí, y se calculan las dos distancias. Se juega la que maximiza

$$d_{\text{rival}} - d_{\text{mío}}$$

Lo importante es que las dos distancias se calculan **después de colocar tu propia ficha**. Si solo minimizas tu camino tienes un bot que corre en línea recta y se deja bloquear como un principiante. Al meter el término del rival, tu ficha se coloca donde además le estorba.

Con lo cual la defensa sale de la fórmula sin que nadie haya escrito nada sobre defender.

## Un error que cuesta caro

La siembra inicial tiene que incluir **todas** las casillas del borde de salida, con coste `0` si ya son tuyas y `1` si están vacías, saltando las del rival.

Sembrar solo las que ya están ocupadas por ti devuelve infinito en cualquier posición donde no tengas ficha tocando tu borde, o sea casi siempre, y eso anula la heurística entera sin dar ningún error.

## Lo que no captura

Los puentes cuentan como distancia `1` cuando en la práctica son `0`, porque son irrompibles. Y tampoco ve las amenazas dobles, donde tienes dos rutas y el rival solo puede cortar una.

Esas dos omisiones son la razón de que un motor entrenado le acabe ganando. También son la razón de que no sea un rival flojo, porque en tableros pequeños la distancia mínima captura buena parte de lo que importa.

## Por qué sirve de ancla

No aprende, no tiene parámetros, y es determinista. O sea que dos mediciones separadas por un mes siguen siendo comparables, cosa que ninguna versión de la propia red puede ofrecer. El protocolo completo está en [[HEX-15 Un metro que no se dilata]].

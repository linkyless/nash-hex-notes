---
title: LEM-replay buffer
tags:
  - lema
---
Una caja donde se guardan las partidas de las últimas iteraciones en vez de tirarlas.

```python
buffer = deque(maxlen=BUFFER_ITERATIONS)
buffer.append(fresh)
all_examples = [ex for piece in buffer for ex in piece]
```

La versión sin buffer genera cincuenta partidas, entrena con ellas, las tira, y repite. Cada ronda se ajusta bastante a un puñado pequeño de partidas muy parecidas entre sí y pierde el acceso a todo lo anterior. Es estudiar para un examen distinto cada semana sin repasar nunca.

El buffer no añade ni un dato nuevo. Lo único que hace es que la distribución sobre la que se entrena sea más ancha y cambie más despacio.

## La pega que casi nadie menciona

Si se dejan las mismas épocas con diez veces más datos, se ha multiplicado por diez el entrenamiento por iteración sin que nadie lo pida. El sobreajuste que se quería evitar entra por la otra puerta.

Conviene razonar en pasos de gradiente por iteración en vez de en épocas, o simplemente bajar las épocas al meter el buffer. Aquí, de cinco a dos.

## Cuánto guardar

Demasiado poco no arregla nada. Demasiado frena el progreso, porque los objetivos viejos vienen de búsquedas guiadas por redes peores y siguen pesando cuando ya no deberían.

AlphaZero usa una ventana de medio millón de posiciones. Aquí, diez iteraciones.

Curiosamente, hay trabajo que reporta menor pérdida de entrenamiento con una ventana de una sola iteración, resultado que los propios autores califican de inesperado. Se mide sobre pérdida y no sobre fuerza de juego, con lo cual conviene cogerlo con pinzas.

## Cuánto aporta

Buffer y simetría metidos juntos, sin nada más, mueven el resultado contra un rival externo de `40/50` a `41/50` y el Elo de `+207` a `+318`.

Mejora real y modesta. No arregla problemas de exploración, que son otra cosa y tienen otra causa.

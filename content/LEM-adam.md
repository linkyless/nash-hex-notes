---
title: LEM-adam
tags:
  - lema
---
Descenso de gradiente con paso adaptativo para cada parámetro por separado.

Guarda dos medias móviles, una del gradiente y otra de su cuadrado.

$$m_t = \beta_1 m_{t-1} + (1-\beta_1) g_t$$
$$v_t = \beta_2 v_{t-1} + (1-\beta_2) g_t^2$$

Y actualiza con

$$\theta_t = \theta_{t-1} - \eta \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \varepsilon}$$

donde los sombreros son una corrección para los primeros pasos, necesaria porque las dos medias empiezan en cero y eso las sesga hacia abajo al principio.

## Qué hace cada parte

**El denominador** es la magnitud típica del gradiente de ese parámetro. Dividir por ella normaliza el paso, con lo cual un parámetro con gradientes consistentemente pequeños avanza igual que uno con gradientes grandes.

Ese es el motivo práctico de que el learning rate deje de tener que calibrarse capa por capa. Aquí vale `0.001` y no lo he tocado en todo el proyecto.

**El numerador** es momento. Promedia direcciones sucesivas, con lo cual las oscilaciones perpendiculares al descenso se cancelan solas y el avance queda más limpio.

Los valores por defecto son $\beta_1 = 0.9$ y $\beta_2 = 0.999$, y prácticamente nadie los cambia.

## Un detalle que aquí no molesta pero conviene saber

En self-play la distribución de los datos cambia entre iteraciones. Las medias móviles tienen memoria larga, del orden de mil pasos para $v$, así que arrastran estadísticas de una distribución anterior.

Con iteraciones cortas eso no llega a notarse. Si algún día los datos cambiaran de golpe, tocaría reiniciar el estado del optimizador.

## Lo que usa AlphaZero

SGD con momento y planificación del learning rate, que con muchos datos suele generalizar algo mejor. Adam converge más rápido al principio, que es lo que interesa cuando lo escaso no es el cómputo sino la paciencia.

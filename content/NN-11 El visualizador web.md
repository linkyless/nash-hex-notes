Cuando terminé de entrenar la red y vi el 97%, sentí que había llegado al final del proyecto. Pero quería enseñarlo y si quieres enseñarlo, necesitas una demo. Algo donde la persona pueda dibujar un dígito y ver que la red funciona en tiempo real.

![[Pasted image 20260426021220.png]]

## La arquitectura

Hay tres piezas que se hablan entre sí. El servidor en Python con Flask carga la red entrenada al arrancar y se queda escuchando. Cuando recibe píxeles de un dibujo, hace forward pass y devuelve las 10 probabilidades. La página HTML es lo que el navegador descarga al visitar la URL. Tiene el canvas, los botones, y los huecos donde aparecerá el resultado. El JavaScript es lo que se ejecuta en el navegador, detecta el ratón sobre el canvas para dibujar, lee los píxeles cuando pulsas predict, los manda al servidor por HTTP, y actualiza la página con la respuesta.

## AVISO

La parte web la vibecodeé en buena parte, sobre todo el CSS. No me arrepiento de ello.

Aprendí bastante bien, sin embargo, cómo funciona Flask (rutas, GET vs POST, `request.get_json`, `jsonify`), cómo se comunica el navegador con el servidor mediante peticiones HTTP y JSON, HTML básico (`<canvas>`, `<button>`, `<div>`, `id` para que JavaScript encuentre los elementos), y JavaScript para canvas (los eventos `mousedown`, `mousemove`, `mouseup`, dibujar líneas con el contexto del canvas, leer píxeles, hacer `fetch` al servidor).

Pero preferí prescindir del CSS y vibecodearlo en ese aspecto. Sé qué hacen las propiedades a un nivel funcional pero no podría escribir el estilo desde cero. Y honestamente, no me importa demasiado. Mi tiempo está mejor invertido en otras cosas, supuse.

Ojito también, otra cosa. El canvas donde dibujas es de $280 \times 280$, no de $28 \times 28$. Si fuera del tamaño de MNIST sería imposible dibujar nada con el ratón. Lo que hago al pulsar predict es crear un canvas temporal en memoria de $28 \times 28$, dibujar el grande dentro del pequeño (el navegador redimensiona automáticamente), y leer los píxeles de ese canvas pequeño. Así puedo dibujar cómodo y la red recibe lo que espera.
También: el reshape al recibir los píxeles en el servidor. La red, después del cambio a mini-batches, espera matrices de , no vectores. Como el visualizador manda solo un dibujo, hago reshape(1, 784) para envolverlo en un batch de tamaño 1, y luego [0] para sacar la primera fila de las predicciones. Pequeño, pero importante.

## Las barras de probabilidades

En lugar de solo mostrar el dígito predicho, mostré las 10 probabilidades como barras horizontales. 

## Recapitulación

El visualizador convierte el proyecto en algo enseñable. Tres piezas: servidor Flask, página HTML, JavaScript. Trabajan juntas mediante peticiones HTTP. La red recibe píxeles, hace forward pass, devuelve probabilidades, y JavaScript las muestra como barras.

Esto cierra el proyecto. Lo siguiente son los conceptos sueltos que aparecen referenciados a lo largo de las notas, en la carpeta de conceptos.
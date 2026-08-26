---
title: HEX-02 Memoria episódica sin estados intermedios
tags:
  - hilo
---

![[computational-graph.png]]

En el proyecto anterior, no quiero larpear de más, pero derivé backpropagation a mano, escribiéndolo en una pizarra que me compré en otoño del año pasado para el segundo semestre del nuevo curso. Las derivadas las traduje en operaciones de numpy, y cuando el coste bajó por primera vez tuve la sensación bastante concreta de haber entendido algo de verdad, aunque no fuera casi nada ni fuera un terrible gran mérito. Fue lo mejor de aquel proyecto con diferencia.

Todo aquello arrancó con el vídeo de 3Blue1Brown, que es precioso y que tiene el efecto secundario de que lo ves una vez y sales convencido de que has entendido cómo funciona una red neuronal si estuviste lo suficientemente concentrado como para terminar el vídeo de esta forma, claro. Como dije en el cuaderno anterior, siempre está la parte que el vídeo no dice, y que tienes que ver tú por tu cuenta. No hay horas suficientes como para explicarlo todo.

Aquí no voy a derivar nada a mano, y me costó un rato aceptarlo.

## ¿Por qué no derivamos nada a mano?

La opción purista es escribirlo todo otra vez sin los llamados **frameworks** que te facilitan la vida. Esto es, hacer lo mismo con las llamadas convoluciones y los gradientes a mano, con la coherencia intelectual del proyecto anterior intacta, y el ego palpitando como debería ser.

La opción práctica though es usar PyTorch, que es un framework que, evidentemente, te facilita la vida, y dedicar ese tiempo a lo que de verdad no sé, que es la búsqueda en árbol y el bucle de aprendizaje, el self-play y toda la paranoia.

Estuve un par de días con esto y al final me convenció un argumento self-invented algo tramposo que creo que es correcto de todas formas. Derivar el gradiente de una convolución no me enseñaría nada nuevo, porque es la misma regla de la cadena que ya derivé para capas densas, solo que con más índices y más sitios donde equivocarme en un signo. 
Así que respiro pacientemente porque no me tengo que preocupar por entender la geometría de... wtf? ¿Una red neuronal en 2D? ¿EN 3D? ¿QUÉ ES ESTO? Da igual, no van a haber errores de signo, pero sí conceptuales, como mucho, intentando entender lo que ya está construido.

![[group-of-computational-nodes.webp]]

> [!tip] PD: El párrafo anterior lo escribí sin enterarme de que más adelante iba a encontrarme con un problema tontísimo que casi me jode todo.


## Tensor vs. Array

Un tensor de PyTorch se parece muchísimo a un array de numpy. Puedes indexarlo igual, multiplicarlo igual, preguntarle la forma igual, y si vienes de numpy la primera impresión es que alguien ha copiado y pegado la library. La diferencia está en que el tensor **se acuerda de lo que le has hecho**. Chunguillo, pero tiene como un historial que se guarda.

Cada operación que aplicas queda registrada, con el resultado y también con la operación misma y también con quién fue el operando. De ahí va saliendo un grafo dirigido donde los nodos son operaciones y las aristas son tensores, que se construye solo mientras tú escribes código que parece aritmética normal.

Eso es el [[LEM-el grafo computacional]] y es el mecanismo entero. Cuando al final llamas a `backward()` sobre un escalar (normalmente el error de la red), PyTorch lo que hace es recorrer ese grafo hacia atrás desde ese escalar hasta las hojas, aplicando la regla de la cadena en cada nodo y dejando en cada peso la derivada correspondiente.

O sea que el forward y el backward son el mismo recorrido, pero en dos direcciones opuestas. El forward construye el camino y el backward vuelve por él repartiendo cositas, podríamos llamarle "culpas" a esas cositas. Por eso tiene sentido que el orden sea rígido y que no puedas hacer backward sin haber hecho forward antes, porque no habría camino por donde volver.

Ya está. Ese es el concepto entero de **autograd**. No sé muy bien qué esperaba encontrarme. Crearé si eso un LEM que lo explique mejor porque lo aprendí bien y está guapo.

## Poner a cero

Después de `backward()`, cada peso tiene su gradiente en `.grad`. Una cosa llamada *optimizador* lo lee, actualiza el peso, y, por alguna razón, el `.grad` no se borra, medio como que se acumula, sin reemplazo. Es decir, en la iteración siguiente el nuevo gradiente se suma al que ya había. 

O sea, si nunca llamas a `zero_grad()`, que es lo que hace que se reinicie esta suma de gradientes acumulada, en el paso $n$ tienes acumulado

$$g_1 + g_2 + \dots + g_n$$

Evidentemente todos entendemos qué significa esa fórmula, así que pasemos a una más difícil.

$$\eta$$

Ese es el learning rate y se pronuncia "eta", es una letra griega, y ya lo vimos en el cuaderno de la red de MNIST. Y bueno, ya que estamos, dejo dicho que es un número pequeño que multiplica al gradiente antes de aplicarlo, y que por defecto vale `0.001` porque alguien lo decidió en algún momento y le funcionó como mismo seguramente se han decidido más de la mitad de cosas en este mundo.

Volviendo a la suma de arriba. La primera vez que la vi asumí que el problema sería la dirección, que apuntaría a cualquier parte. Pues no exactamente. La suma de los gradientes de varios lotes es, salvo un factor de escala, el gradiente del conjunto entero de esos lotes, así que la dirección apunta a un sitio bastante razonable y podríamos decir que está bien.

Lo que revienta es la magnitud, que crece linealmente con el número de pasos. En el paso 500th estás dando una zancada quinientas veces más larga de lo que $\eta$ pretendía, y la pérdida se va a hacer gárgaras llevándose la red por delante.

$g_1$, además, se calculó con unos pesos que ya no existen y sigue y sigue sumando en el paso 500th con exactamente el mismo peso que en el 2nd, muy convencido, informando sobre un paisaje que llevas aproximadamente 498 pasos deformando.

Pero en realidad investigué y es una técnica esta idea de "acumular" gradientes. Si quieres simular un lote grande y no te cabe en la memoria de la tarjeta, haces varios forward y backward sin poner a cero y llamas al optimizador una sola vez al final. La diferencia entre el bug y la "maravillosa técnica" es dónde colocas una línea. Así que cuidado.

```python
# bucle de training
optimizer.zero_grad()                      # 1. Poner a cero (romper la memoria episódica)
policy_logits, values_pred = network(boards_t)
total_loss = policy_loss + value_loss
total_loss.backward()                      # 2. Recorrer el grafo hacia atrás
optimizer.step()                           # 3. Actualizar los pesos
```

## Qué me llevo de aquí

Poco, y me parece bien. PyTorch es mecánico. Un tensor, un módulo que hereda de `nn.Module`, un optimizador, y un bucle de cinco pasos que se escribe una vez y no se vuelve a tocar en todo el proyecto.

Lo único que hay que entender de verdad es que el grafo existe. Por eso más adelante, cuando consulte la red desde dentro del árbol, voy a envolver la llamada en `torch.no_grad()`. Ahí no entreno, y construir un grafo que nadie va a recorrer sale caro cuando lo haces cientos y cientos de miles de veces.

```python
# dentro de la expansión de nodos en mcts.py:
tensor = torch.from_numpy(curr_board).float()

with torch.no_grad():
    policy_logits, value = self.network(tensor)
```

Efecto secundario agradable, los tensores que salen de ese bloque no arrastran gradiente, así que se pueden convertir a numpy directamente sin ceremonia. Cool enough.

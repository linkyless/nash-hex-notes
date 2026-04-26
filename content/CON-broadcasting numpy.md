Broadcasting es la forma en la que numpy maneja operaciones entre arrays de formas distintas. La idea es que si las dimensiones son compatibles, numpy "extiende" automáticamente el array más pequeño para que cuadre con el más grande, sin tener que escribir bucles.

El ejemplo más simple es sumar un vector $(3,)$ a un escalar. Numpy entiende que el escalar se suma a cada elemento del vector. No necesitas escribir un bucle.

Más interesante: sumar una matriz $(32, 10)$ con un vector $(10,)$. Numpy interpreta que el vector se suma a cada fila de la matriz. Es como si el vector fuera de $(32, 10)$ con todas las filas iguales, pero como tal no queremos gastar memoria.

En el proyecto aparece todo el rato. Cuando hago $z = Wa + b$ con un batch, $b$ es un vector $(10,)$ y se suma automáticamente a cada fila de la matriz $(32, 10)$ que sale de $Wa$. Sin broadcasting, tendría que repetir el bias 32 veces a mano.

También en softmax con `keepdims=True`. Tengo una matriz $(32, 10)$ y quiero dividir cada fila por su suma. Calculo las sumas con forma $(32, 1)$ (gracias a `keepdims`), y al dividir $(32, 10) / (32, 1)$ broadcasting expande automáticamente la columna a 10 columnas. Cada fila se divide por su propia suma.

Las reglas exactas de broadcasting son un poco intrincadas, pero la regla práctica es: las dimensiones cuadran si son iguales o si una de ellas es 1. Si no cuadran, numpy se queja y tienes que reformar con `reshape` o `keepdims`.
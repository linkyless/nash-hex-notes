---
title: LEM-union-find
tags:
  - lema
---
Mantiene una partición de $n$ elementos con dos operaciones, unir dos conjuntos y preguntar si dos elementos están en el mismo.

El código de este proyecto es una traducción directa del que llevo años usando en competitiva, aprendido el día que me lo dio un profesor (AV) en una clase online en el server de la Olimpiada Informática Española.

```cpp
struct ufds {
    int numSets;
    vector<int> p;

    ufds(int N) : numSets(N), p(N, -1) {}

    int find(int i){
        return (p[i] < 0) ? i : p[i] = find(p[i]);
    }

    bool related(int i, int j){
        return find(i) == find(j);
    }

    void join(int i, int j){
        int x = find(i), y = find(j);
        if (x == y) return;
        if (p[x] < p[y]) swap(x, y);
        p[y] += p[x];
        p[x] = y;
        numSets--;
    }
    int size(int i){
        return -p[find(i)];
    }
};
```

## El truco del vector único

Un nodo o es raíz o no lo es, nunca las dos cosas, y eso permite meter los dos casos en el mismo array. Si `p[i]` es negativo, `i` es raíz y ese negativo guarda el tamaño del conjunto. Si es positivo, es el índice del padre.

Con eso, `size` es leer y negar, la unión por tamaño es comparar dos negativos, y la inicialización es rellenar de `-1`. Ninguna estructura auxiliar.

La línea del `find` hace dos cosas a la vez. Sube hasta la raíz y de paso reescribe `p[i]` con el resultado, así que la siguiente consulta desde ese nodo llega en un salto. Eso es compresión de caminos, metida dentro de una asignación dentro de un ternario dentro de un return.

Combinada con unión por tamaño, cada operación cuesta $O(\alpha(n))$, donde $\alpha$ es la inversa de Ackermann y vale menos de cinco para cualquier entrada que quepa en este universo. En la práctica es constante.

## En Hex

Dos estructuras separadas, una por jugador, cada una con las casillas más cuatro nodos de borde. Colocar une la casilla con sus vecinos del mismo color y con el borde que toque, y la victoria se consulta preguntando si dos nodos de borde comparten raíz.

Detalle. En Python la recursión del `find` es cara y tiene límite de profundidad. Para tableros grandes conviene la versión iterativa.

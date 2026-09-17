# Reporte EDA — Operación Dino Crash
**Analista:** Kevin Andrick Suárez García

## 1. Problema y dataset (Misión 1)

Antes de meternos a correr modelos, lo primero es definir bien contra qué nos estamos enfrentando en cada escenario del juego del dinosaurio.

### P1 — ¿Morirá en el siguiente frame?
- **Variable objetivo (Y):** La columna `died`. Es binaria (1 si se acaba el juego justo ahí, 0 si sigue la jugada).
- **Variables de entrada (X):** Pediría mínimo cosas como la velocidad actual (`speed`), la distancia real al obstáculo (`dist_obstacle`), el tipo de bicho o cactus que viene (`obstacle_type`), si el dinosaurio ya va saltando (`jump`), y el tiempo que lleva corriendo la partida (`time_ms`).
- **Granularidad:** Aquí sí o sí necesitamos un registro por cada frame (cada 16 milisegundos). El golpe pasa en fracciones de segundo, así que ver esto resumido por partida no nos serviría de nada.
- **Tamaño mínimo:** Unas 100 partidas completas estarían bien para empezar, porque los momentos de muerte son muy pocos comparados con todo el tiempo que pasa uno corriendo.
- **Riesgo:** Si definimos mal esto y mezclamos sesiones o frames sin sentido, el modelo va a aprender pura basura y va a fallar en cuanto cambie ligeramente la velocidad del juego.

### P2 — ¿Cuántos puntos alcanzará esta partida al morir?
- **Variable objetivo (Y):** Un `score_final` numérico entero.
- **Variables de entrada (X):** Aquí ya no importan los frames sueltos, sino métricas globales de la partida, como la velocidad máxima que alcanzó (`max_speed`), cuántos obstáculos aéreos esquivó (`total_birds`), qué tan rápido reaccionaba en promedio, y el tiempo total de vida.
- **Granularidad:** Un resumen por partida completa (una sola fila por cada intento).
- **Tamaño mínimo:** Unas 300 o 500 partidas para que el algoritmo realmente encuentre patrones entre el comportamiento y la puntuación final.
- **Riesgo:** Si intentamos predecir esto queriendo usar datos cuadro por cuadro sin agruparlos, el modelo se va a marear con las dimensiones y no sabrá qué escala arrojar.

### P3 — ¿Qué tipo de obstáculo viene próximo?
- **Variable objetivo (Y):** `next_obstacle_type`, que es categórica (`none`, `cactus_small`, `cactus_large`, `bird`).
- **Variables de entrada (X):** El puntaje actual (`score`), la velocidad del momento, cuánto tiempo ha pasado desde el último obstáculo, y el `time_ms`.
- **Granularidad:** Se evalúa por intervalo o cada vez que hay un objeto cerca en pantalla.
- **Tamaño mínimo:** Unas 200 partidas para asegurarnos de que alcancen a salir los pájaros (que son los que menos aparecen).
- **Riesgo:** Si medimos en frames muertos donde no hay nada cerca, el modelo se la va a pasar prediciendo la clase `none` y no nos avisará cuando de verdad venga una amenaza.

---

## 2. Diccionario y muestra (Misión 2)

Dándole una leída a los datos de la sesión 7 que nos pasaron:
- **Patrón en la muerte (frame 82):** Si te fijas, el choque pasa justo cuando la distancia al obstáculo (`dist_obstacle`) cae a un punto crítico muy bajo y el salto no se activó a tiempo. Básicamente el bicho se lo traga porque ya no hay margen de reacción.
- **¿Sirve el `score` para predecir si vas a morir en el siguiente frame?:** La neta no como causa directa. El score sube solito conforme pasa el tiempo, no es que el puntaje alto mate al dinosaurio. Usarlo como predictor principal solo metería ruido falso.
- **Lo que le falta a este dataset:** Se siente que faltan datos clave de la física del juego, como la altura exacta en la que va el dinosaurio en el aire, si va agachado, o el ligero retraso (*lag*) del teclado cuando uno presiona la tecla.
- **Sobre la columna `died`:** Tal como está planteada, solo sirve como una etiqueta de "fin de juego" para voltear atrás y ver qué pasó, pero no nos dice mucho por sí sola si no la cruzamos con la distancia.

---


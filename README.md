# T02 · ¿9z gana este mapa?

**Integrantes:** Nahuel Ishikawa · Alejandro Martínez
**IA declarada:** deepseek
**Pista:** Neurona
**Publicación en LinkedIn:** <link>

---

## 1. Formulación y PEAS

**La pregunta.** Para cada caso, ¿9z gana este mapa? **SÍ** significa: 9z gana el mapa. **NO** significa: 9z pierde el mapa. El sistema decide durante el veto, antes de que el mapa se juegue.

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | Armar el veto para que 9z juegue mapas que gana. Métrica operativa: minimizar el error más caro (falso positivo: dejar en el veto un mapa que se pierde). Se reportan exactitud, precisión, sensibilidad y especificidad. |
| Entorno | Mapas jugados por 9z en torneos; datos históricos pre‑partido (ranking, pistolas, mapa elegido). |
| Actuadores | Respuesta SÍ/NO al analista y al capitán para el veto. |
| Sensores | Lectura de las 3 variables pre‑partido del caso (CSV). |

**Objetivo y métrica.** El objetivo es ganar mapas en el veto; la métrica es la calidad de la clasificación. No son lo mismo: no se optimiza la cantidad de SÍ ni el tiempo de espera, sino la capacidad de no dejar mapas perdedores en el veto.

## 2. El sistema

**Cómo decide.** Perceptrón con 3 entradas (R, P, M):

\[
\text{score} = 1\cdot R + 0\cdot P + 10.5\cdot M - 5
\]

\[
\text{SÍ si score} > 0;\quad \text{NO si score} \le 0
\]

Interpretación: si 9z eligió el mapa (M=1), gana si R ≥ −5. Si no lo eligió (M=0), gana si R ≥ 6.

**Variables que usa y por qué.** R = diferencia de ranking (positivo = 9z mejor rankeado). P = rondas de pistola ganadas (%). M = mapa elegido por 9z (0 = no, 1 = sí). M se conoce en el mismo acto del veto: es el mapa candidato que el analista y el capitán están evaluando; el sistema no predice M, lo recibe y responde “si 9z juega este mapa, ¿lo gana?”. Se descarta “rondas ganadas en el primer tiempo” porque no existe al decidir (leakage). El perceptrón se entrenó con las 3 variables pre‑partido; el peso aprendido para P fue 0, por lo que la decisión efectiva depende de R y M. No es un descarte manual: es lo que el modelo aprendió de los datos.

**Cómo se corre en vivo.** 1) Tomar las 3 variables del caso: R, P, M. 2) Calcular `score = 1·R + 0·P + 10.5·M − 5`. 3) Si `score > 0` → SÍ; si `score ≤ 0` → NO. Se puede hacer a mano, en planilla o en Colab con la fórmula.

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | R = 15, P = 50, M = 1 → score = 15 + 0 + 10.5 − 5 = 20.5 > 0 | SÍ (9z gana) |
| B | R = −10, P = 70, M = 0 → score = −10 + 0 + 0 − 5 = −15 ≤ 0 | NO (9z pierde) |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 7 | FN = 0 |
| **Real NO** | FP = 0 | VN = 7 |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 3 | FN = 1 |
| **Real NO** | FP = 0 | VN = 2 |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | 14/14 = 100% | 5/6 = 83,3% |
| Precisión | 7/7 = 100% | 3/3 = 100% |
| Sensibilidad | 7/7 = 100% | 3/4 = 75% |
| Especificidad | 7/7 = 100% | 2/2 = 100% |

**Entrenamiento contra prueba.** La exactitud baja de 100% a 83,3% por un único FN en test (caso 15: R=−4, M=0, real=1). No hay FP en ninguno de los dos conjuntos. La caída es esperable al pasar de 14 a 6 casos y no indica sobreajuste grave: el modelo tiene 2 parámetros efectivos (R y M) y 1 peso en 0.

## 4. El error más caro

El falso positivo (decir SÍ y que el mapa se pierda) es el error más caro: queda en el veto un mapa perdedor. Según la ficha, eso significa “dejar en el veto un mapa que se pierde”. Para cometer menos ese error: descartamos “rondas del primer tiempo” por leakage, ajustamos pesos y umbral para que en entrenamiento no haya ningún FP, aceptamos 1 FN en test (caso 15) antes que arriesgar un FP, y probamos un perceptrón con P con peso 1 que daba 6/6 en test pero 1 FP en entrenamiento (caso 11); lo rechazamos porque viola la prioridad del error más caro.

## 5. ¿Comparado contra qué?

**Línea base trivial.** En entrenamiento (14 casos: 7 SÍ reales, 7 NO reales), decir siempre SÍ o siempre NO acierta 7/14 = 50,0%. Nuestra neurona: 14/14 = 100% (+50 pp). En prueba (6 casos: 4 SÍ reales, 2 NO reales): siempre SÍ acierta 4/6 = 66,7% pero con 0% de especificidad (2 FP: casos 16 y 20); siempre NO acierta 2/6 = 33,3% con 0% de sensibilidad; nuestra neurona: 5/6 = 83,3% de exactitud, 100% de precisión y especificidad (0 FP). Supera ambas líneas base.

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | 20 casos inventados de mapas de 9z (`datos/T02.csv`), divididos de forma fija en 14 de entrenamiento (casos 1–14) y 6 de prueba (casos 15–20). Usamos las 3 variables disponibles antes del veto (R, P, M); el perceptrón aprendió peso 0 para P. Descartamos “rondas del primer tiempo” por no existir al momento de decidir (leakage). |
| ¿Qué métrica? | Precisión y Especificidad como métricas prioritarias para controlar el error más caro (FP = dejar en el veto un mapa que se pierde), acompañadas de Exactitud y Sensibilidad. |
| ¿Comparado contra qué? | Contra la línea base trivial que responde siempre la misma clase sin mirar los datos (50,0% en entrenamiento; en prueba 66,7% diciendo siempre SÍ pero con 0% de especificidad y 2 FP, o 33,3% diciendo siempre NO). Nuestro sistema logra 100% en entrenamiento y 83,3% en prueba con 0 FP. |
| ¿Quién lo midió? | Primero nuestro grupo sobre los 6 casos separados de prueba (15–20), y luego de forma externa e independiente el bot evaluador (`@Evaluador_iia_uade_bot`) y el profesor en la mesa con un caso nuevo que no vimos. |

## 6. Riesgo: el camino completo

Datos: 14 casos de train y 6 de test, inventados; pueden no reflejar el meta actual de 9z. Modelo: perceptrón lineal simple; no captura interacciones complejas entre variables. Decisión: SÍ/NO para el veto. Consecuencia: FP = mapa perdedor en el veto; FN = vetar un mapa ganable. El modelo prioriza no tener FP. Un riesgo concreto: si el meta cambia y el ranking deja de reflejar la forma actual, el sistema puede decir SÍ a un mapa que 9z ya no domina, y ese mapa queda en el veto.

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | “Tengo 20 casos con R, P, M y resultado. Quiero un perceptrón pre‑partido sin leakage.” | Fórmula y pesos. | Usar primer tiempo (leakage). |
| 2 | “Probá un perceptrón con las 3 variables que dé 6/6 en test.” | Pesos con P=1. | Deja 1 FP en train (caso 11); viola la prioridad del error más caro. |
| 3 | “¿Un solo perceptrón puede dar 14/14 y 6/6?” | Respuesta: no, el caso 15 es linealmente inseparable del entrenamiento. | Agregar capas ocultas por overfitting (14 train, 6 test). |
| 4 | “El evaluador marcó que no está claro de dónde sale M y cuándo se conoce. ¿Cómo lo justificamos?” | M es el mapa que 9z elige en el mismo veto; es una acción del decisor, no una variable futura. | — |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | 8,5/10 | No estaba claro de dónde sale M ni cuándo se conoce; faltaban instrucciones para correr un caso nuevo con ejemplos. | Se agregó la justificación de M en la sección 1 (M es el mapa que 9z elige en el veto, conocido en el acto del veto) y se agregaron los ejemplos 1 y 2 paso a paso en la sección 2. |
| 2 | — | — | — |

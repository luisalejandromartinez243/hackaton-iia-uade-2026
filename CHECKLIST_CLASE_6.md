# Checklist Crítica de la Clase 6 · Guía Completa + Aplicación a `T02 (9z)`

> **Referencia de Cátedra:** Apunte Oficial *«Clase 6 · Unidad 3 — AI snake oil: qué se vende y qué funciona»* (Narayanan & Kapoor, 2024; Raji et al., 2022; Wong et al., 2021) y **Criterio E de `BASES.md`** (**1 punto** + **Tope de 7 puntos** si no se compara contra la línea base trivial).

---

## 1. ¿Qué es la Checklist de la Clase 6 y para qué sirve?

En la **Clase 6** (*AI Snake Oil y Evaluación Crítica*), la cátedra enseña que nunca hay que dar por sentado que un sistema de IA funciona solo porque el creador dice *"tiene 85% de éxito"* (**falacia de suposición de funcionalidad**).

Para auditar cualquier sistema de IA (tanto el que vende una empresa como **nuestro propio sistema en el Hackatón**), siempre hay que hacerle **4 preguntas obligatorias**:

1. **¿Qué datos?**
2. **¿Qué métrica?**
3. **¿Comparado contra qué?**
4. **¿Quién lo midió?**

---

## 2. Explicación de cada una de las 4 preguntas (En Criollo + Qué mira el Evaluador)

### Pregunta 1: ¿Qué datos?
* **Qué pregunta en criollo:** ¿De dónde salieron los casos con los que entrenaste y probaste el sistema? ¿Cuántos son? ¿Estaban separados los de entrenamiento y los de prueba? ¿Usaste solo datos que existían antes de tomar la decisión o hiciste trampa con datos del futuro (*data leakage*)? ¿Qué sesgos tienen?
* **Qué tenés que responder en `T02`:**
  * Son **20 casos inventados** de mapas jugados por 9z (`datos/T02.csv`), divididos de forma fija en **14 de entrenamiento (casos 1–14)** y **6 de prueba (casos 15–20)**.
  * Usamos **solo las 3 variables disponibles al momento de armar el veto** (`Diferencia de ranking`, `% de rondas de pistola en el torneo` y `Mapa elegido por 9z`).
  * **Descartamos** `Rondas ganadas en el primer tiempo (de 12)` porque en el momento del veto el partido todavía no empezó (usarla rompería la **validez interna** por fuga de datos).

---

### Pregunta 2: ¿Qué métrica?
* **Qué pregunta en criollo:** ¿Con qué número mediste si el sistema anda bien? ¿Elegiste la métrica correcta para el **error más caro** de tu problema o miraste solo la Exactitud general?
* **Qué tenés que responder en `T02`:**
  * Medimos **Exactitud, Precisión, Sensibilidad y Especificidad** sobre las matrices de entrenamiento y prueba.
  * Pero como en nuestro problema el **error más caro es el Falso Positivo** (*decir SÍ cuando era NO = dejar abierto en el veto un mapa que 9z pierde*), nuestras métricas principales de decisión son la **Precisión** ($\text{VP}/(\text{VP}+\text{FP})$) y la **Especificidad** ($\text{VN}/(\text{VN}+\text{FP})$), donde logramos **$100\%$ ($0$ Falsos Positivos)** tanto en entrenamiento como en prueba, con **$83,3\%$ de Exactitud** y **$75,0\%$ de Sensibilidad** en prueba.

---

### Pregunta 3: ¿Comparado contra qué? *(¡OJO: Evita el Tope de 7 puntos!)*
* **Qué pregunta en criollo:** Decir *"mi neurona acertó el 83,3%"* suelto en el aire no significa nada si no lo comparás contra una **línea base trivial** (una "neurona tonta" que responde siempre lo mismo sin mirar ningún dato).
* **Qué tenés que responder en `T02`:**
  * **En Entrenamiento (14 casos: 7 SÍ reales y 7 NO reales):** La línea base trivial (decir siempre SÍ o siempre NO) acierta **$7 / 14 = 50,0\%$**. Nuestra neurona saca **$100\%$ ($14/14$)** (+50 puntos porcentuales).
  * **En Prueba (6 casos: 4 SÍ reales y 2 NO reales):**
    * Si la línea base trivial dice **siempre SÍ** (la clase más frecuente): acierta **$4 / 6 = 66,7\%$ de Exactitud**, pero tiene **$0\%$ de Especificidad** porque se come los 2 mapas perdidos como Falsos Positivos (Casos 16 y 20).
    * Si la línea base trivial dice **siempre NO**: acierta solo **$2 / 6 = 33,3\%$ de Exactitud** (y $0\%$ de Sensibilidad).
    * **Nuestra neurona saca $83,3\%$ ($5/6$) de Exactitud en prueba** y **$100\%$ de Precisión y Especificidad ($0$ Falsos Positivos)**, superando a ambas líneas base.

---

### Pregunta 4: ¿Quién lo midió?
* **Qué pregunta en criollo:** ¿Lo midió únicamente el propio creador del sistema con sus propios casos (como en el caso de la Clase 6 donde el fabricante *Epic* decía que su detector de sepsis tenía $0,83$, pero cuando lo midió una universidad independiente dio $0,63$), o hubo una auditoría externa con casos separados?
* **Qué tenés que responder en `T02`:**
  * **Medición interna:** La hicimos nosotros (los diseñadores del grupo) respetando la separación estricta de los **6 casos de prueba (15–20)** que no se usaron para elegir los pesos ni el sesgo.
  * **Auditoría y validación externa:** La realizan de forma independiente el **agente evaluador de la cátedra (`@Evaluador_iia_uade_bot`)** y **el docente en vivo en la mesa rodante** probando el sistema con un caso nuevo fuera de nuestra ficha.

---

## 3. Tabla Lista para Copiar y Pegar en la Sección 5 del `README.md`

```markdown
| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | 20 casos inventados de mapas de 9z (`datos/T02.csv`), divididos de forma fija en **14 de entrenamiento (casos 1–14)** y **6 de prueba (casos 15–20)**, usando solo las 3 variables disponibles antes del veto ($x_1, x_2, x_3$) y descartando $x_4$ (rondas del primer tiempo) por no existir al momento de decidir. |
| ¿Qué métrica? | **Precisión y Especificidad** como métricas prioritarias para controlar el error más caro (Falsos Positivos = dejar en el veto un mapa que se pierde), acompañadas de **Exactitud y Sensibilidad**. |
| ¿Comparado contra qué? | Contra la **línea base trivial** que responde siempre la misma clase sin mirar los datos ($50,0\%$ de exactitud en entrenamiento; en prueba $66,7\%$ diciendo siempre SÍ pero con $0\%$ de especificidad y 2 FP, o $33,3\%$ diciendo siempre NO). Nuestro sistema logra $100\%$ en entrenamiento y $83,3\%$ en prueba con $0$ FP. |
| ¿Quién lo midió? | Primero nuestro propio grupo sobre los 6 casos separados de prueba (15–20), y luego de forma externa e independiente el bot evaluador (`@Evaluador_iia_uade_bot`) y el profesor en la mesa con un caso nuevo que no vimos. |
```

---

## 4. Cómo responder en la Mesa si el Profesor les pregunta por la Checklist de la Clase 6 (Sin IA)

> **Si el profesor pregunta:** *"¿Para qué sirve la checklist de las 4 preguntas de la Clase 6 y qué encontraron al aplicarla a su neurona?"*
>
> **Respuesta en 20 segundos:**
> *"Sirve para no comerse el humo o 'snake oil' de un porcentaje suelto.
> 1) En **¿Qué datos?** verificamos que los 6 casos de prueba estén separados de los 14 de entrenamiento y sacamos las rondas del primer tiempo para no tener fuga de datos.
> 2) En **¿Qué métrica?** no nos quedamos solo con la Exactitud ($83,3\%$), sino que miramos **Precisión y Especificidad ($100\%$)** porque en el veto el Falso Positivo es el error más caro.
> 3) En **¿Comparado contra qué?** demostramos que nuestro $83,3\%$ le gana a la línea base trivial (que saca $66,7\%$ diciendo siempre SÍ pero comiéndose todos los mapas perdidos con $0\%$ de especificidad).
> 4) Y en **¿Quién lo midió?** distinguimos nuestra medición sobre los casos 15 a 20 de la validación externa que hace usted ahora en la mesa con un caso nuevo."*

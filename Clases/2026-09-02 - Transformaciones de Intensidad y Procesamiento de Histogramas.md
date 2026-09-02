---
titulo: "Clase 5 — Transformaciones de Intensidad y Procesamiento de Histogramas"
materia: Visión Artificial
tipo: clase
fecha: 2026-09-02
semana: 4
tags:
  - visión-artificial
  - clase
  - transformaciones-espaciales
  - corrección-gamma
  - histogramas
  - ecualización
  - python
  - opencv
---

# Clase 5 — Transformaciones de Intensidad y Procesamiento de Histogramas

> [!info] Fecha
> 📅 Miércoles, 2 de septiembre de 2026 — Semana 4

## 📝 Temas vistos

1. **Núcleo del procesamiento en el dominio espacial (Operaciones Puntuales)**:
   - Mapeo directo píxel a píxel con ventana $1 \times 1$:
     $$ s = T(r) $$
     donde $r$ es la intensidad de entrada del píxel y $s$ es la intensidad procesada de salida, con $r, s \in [0, L-1]$.
2. **Transformaciones de Intensidad sobre la Imagen Lunar (`data.moon()`)**:
   - **Inversión Fotográfica / Negativo ($s = (L-1) - r$)**: Resalta cráteres y detalles oscuros invirtiendo la escala.
   - **Transformación Logarítmica ($s = c \log(1 + r)$)**: Expande intensidades oscuras haciendo visibles detalles tenues en el terreno lunar.
   - **Corrección Gamma ($s = c \cdot r^\gamma$)**: Función modular `gamma_transform` probada con $\gamma = 0.5$ (aclarado), $\gamma = 1.0$ (identidad) y $\gamma = 2.0$ (oscurecido de sombras).
   - **Estiramiento de Contraste por Percentiles**: Selección de umbrales robustos con `np.percentile(img, p)` y truncamiento seguro con `np.clip`.
3. **Histogramas de Imagen y Ecualización**:
   - Diagnóstico visual de iluminación y contraste mediante la función de distribución de frecuencias.
   - Ecualización de histograma basada en la Función de Distribución Acumulada (CDF) para maximizar el rango dinámico.

---

## 🔍 Corrección y Clarificación Teórica

> [!important] Aclaración sobre la Transformación Logarítmica
> - **Efecto real:** La función logaritmo **expande** las intensidades bajas (tonos oscuros), haciéndolas más distinguibles y brillantes, y **comprime** las intensidades altas (tonos brillantes).
> - Se observa claramente en la imagen procesada de la Luna: el fondo negro y el polvo lunar gris se vuelven claros, mientras que los picos brillantes no deslumbran.

---

## 🧮 Ejemplos y Operaciones Resueltas Paso a Paso

### 📌 1. Cálculo de $c_{log}$ para la Luna
$$ c_{log} = \frac{255}{\ln(1 + \max(img))} $$
Para una imagen de 8 bits con $\max(img) = 255$:
$$ c_{log} = \frac{255}{\ln(256)} \approx \frac{255}{5.545} \approx \mathbf{45.98} $$

---

### 📌 2. Corrección Gamma ($s = 255 \times (r / 255)^\gamma$)
Para un píxel con intensidad $r = 64$ ($norm = 64/255 \approx 0.2509$):
* **$\gamma = 0.5$ (Aclarar):** $s = 255 \times (0.2509)^{0.5} = 255 \times 0.5009 \approx \mathbf{128}$
* **$\gamma = 1.0$ (Identidad):** $s = 255 \times (0.2509)^{1.0} \approx \mathbf{64}$
* **$\gamma = 2.0$ (Oscurecer):** $s = 255 \times (0.2509)^{2.0} = 255 \times 0.0630 \approx \mathbf{16}$

---

### 📌 3. Estiramiento de Contraste con Percentiles
Para aislar y maximizar el rango de intensidades intermedias entre los percentiles $r_{\min} = P_{40}$ y $r_{\max} = P_{50}$:

$$ s = \text{clip}\left( \frac{r - r_{\min}}{r_{\max} - r_{\min}} \times 255, 0, 255 \right) $$

- Todo píxel con $r \le r_{\min}$ se satura a **0 (negro)**.
- Todo píxel con $r \ge r_{\max}$ se satura a **255 (blanco)**.
- El intervalo estrecho $[r_{\min}, r_{\max}]$ se expande linealmente ocupando todo el rango $[0, 255]$ (generando un efecto de binarización / alto contraste localizado).

---

## 🖼️ Resultados Visuales de la Sesión

### 1. Imagen Original (`skimage.data.moon()`)
![[transformacion-luna-original.png]]

### 2. Inversión Fotográfica / Negativo
![[transformacion-luna-negativo.png]]

### 3. Transformación Logarítmica
![[transformacion-luna-logaritmica.png]]

### 4. Comparativa de Corrección Gamma ($\gamma = 0.5, 1.0, 2.0$)
![[transformacion-luna-gamma-comparacion.png]]

### 5. Estiramiento de Contraste con Percentiles
![[transformacion-luna-estiramiento-percentiles.png]]

---

## 🔗 Notas relacionadas

- 🧠 Conceptos:
  - [[Transformaciones de Intensidad Espacial]]
  - [[Histogramas y Ecualización de Imagen]]
  - [[Desbordamiento y Normalización de Imágenes]]
  - [[Resolución de Imagen]]
- 💻 Código:
  - [[Transformaciones de Intensidad y Histogramas - Python]]
- 📅 Sesión previa:
  - [[2026-09-01 - Operaciones Aritméticas y Lógicas entre Imágenes]]

## 📌 Pendientes / tareas

- [ ] Comparar el estiramiento de percentiles amplios ($P_2$ a $P_{98}$) frente al estiramiento estrecho ($P_{40}$ a $P_{50}$).
- [ ] Implementar CLAHE sobre la imagen lunar para realzar el relieve de los cráteres.

## 🏷️ Etiquetas

#visión-artificial #clase #transformaciones-espaciales #corrección-gamma #histogramas #ecualización #python #opencv

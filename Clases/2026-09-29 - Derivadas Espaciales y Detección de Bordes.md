---
titulo: "Clase 10 — Derivadas Espaciales y Detección de Bordes"
materia: Visión Artificial
tipo: clase
fecha: 2026-09-29
semana: 8
tags:
  - visión-artificial
  - clase
  - derivadas-espaciales
  - detección-de-bordes
  - gradiente
  - laplaciano
  - python
  - opencv
---

# Clase 10 — Derivadas Espaciales y Detección de Bordes

> [!info] Fecha
> 📅 Martes, 29 de septiembre de 2026 — Semana 8

## 📝 Temas vistos

1. **Evolución del procesamiento espacial**:
   - De operaciones puntuales (brillo/contraste) y filtros de suavizado (paso bajo/promedios) pasamos a la **detección de cambios geométricos y estructurales** en la imagen mediante operadores diferenciales (filtros paso alto).
2. **Naturaleza discreta de la derivada**:
   - En cálculo continuo, la derivada es un límite infinitesimal:
     $$ \frac{df}{dx} = \lim_{\Delta x \to 0} \frac{f(x + \Delta x) - f(x)}{\Delta x} $$
   - En una imagen digital discreta, la distancia mínima posible entre muestras es de **$1$ píxel** ($\Delta x = 1$), por lo que las derivadas se calculan mediante **diferencias finitas entre vecinos**.
3. **Primera Derivada Discreta ($d[i]$)**:
   - Aproximación hacia adelante:
     $$ d[i] = p[i+1] - p[i] $$
   - **Propósito:** Mide la velocidad o tasa de cambio de la intensidad. Identifica las transiciones entre regiones oscuras y claras.
4. **Segunda Derivada Discreta ($f''(x)$)**:
   - Derivada de la primera derivada (diferencia de diferencias):
     $$ f''(x) = (f(x+1) - f(x)) - (f(x) - f(x-1)) = f(x+1) + f(x-1) - 2f(x) $$
   - **Propósito:** Mide la curvatura o aceleración del cambio de intensidad.
5. **Comportamiento ante estructuras geométricas de borde**:
   - **Zona plana:** Ambas derivadas valen estrictamente $0$.
   - **Escalón (*Step edge*):** Salto abrupto de intensidad. La 1ra derivada produce un pico simple grueso; la 2da derivada produce un **doble pico con signos opuestos** que cruza por cero (*zero-crossing*).
   - **Rampa (*Ramp edge*):** Transición suave y gradual. La 1ra derivada produce un valor constante no nulo (borde grueso); la 2da derivada es cero durante la rampa y produce picos solo en el inicio y fin.
6. **Sensibilidad al Ruido:**
   - La derivación amplifica agresivamente las altas frecuencias. La 2da derivada es **extremadamente sensible al ruido**, por lo que casi siempre requiere un filtrado gaussiano previo (originando el operador *Laplaciano del Gaussiano* o LoG).

---

## 🧮 Ejemplo Numérico Resuelto Paso a Paso

Supongamos la siguiente fila de píxeles:
$$ P = [30, 30, 30, 100, 150, 150, 150] $$

### 📌 1. Cálculo de la Primera Derivada: $d[i] = p[i+1] - p[i]$

> [!question] ¿Por qué pasa de 0 a 70?
> No es que el píxel 30 se convierta en 70. Lo que se calcula es la **resta entre el píxel que sigue y el actual**:
> * En $i=0$: $30 - 30 = \mathbf{0}$ (sin cambio).
> * En $i=1$: $30 - 30 = \mathbf{0}$ (sin cambio).
> * En $i=2$: El píxel salta de $30$ a $100$. La diferencia es $100 - 30 = \mathbf{70}$ (¡aparece el inicio del borde!).
> * En $i=3$: El píxel sube de $100$ a $150$. La diferencia es $150 - 100 = \mathbf{50}$ (continúa subiendo).
> * En $i=4$: $150 - 150 = \mathbf{0}$ (la meseta se estabiliza).
> * En $i=5$: $150 - 150 = \mathbf{0}$ (plano).

$$ d_1 = [0, 0, 70, 50, 0, 0] $$

---

### 📌 2. Cálculo de la Segunda Derivada (sobre el resultado previo)
Diferencia entre elementos consecutivos de la primera derivada:
* $0 - 0 = \mathbf{0}$
* $70 - 0 = \mathbf{70}$ (inicio del ascenso: aceleración positiva).
* $50 - 70 = \mathbf{-20}$ (desaceleración de la subida).
* $0 - 50 = \mathbf{-50}$ (frenado al llegar a la cima plana).
* $0 - 0 = \mathbf{0}$

$$ d_2 = [0, 70, -20, -50, 0] $$

---

## 📊 La "Matriz de la Verdad": 1ra vs 2da Derivada

| Escenario / Criterio | Primera Derivada ($\nabla f$) | Segunda Derivada ($\nabla^2 f$) |
| :--- | :--- | :--- |
| **Zona Plana (Constante)** | **Cero ($0$)** | **Cero ($0$)** |
| **Borde Escalón (*Step*)** | Pico único no nulo (borde grueso) | **Doble pico (signo positivo y negativo)** |
| **Borde Rampa (*Ramp*)** | Constante durante la subida (borde ancho) | **Cero durante la rampa**; picos solo en bordes |
| **Cruce por Cero (*Zero-crossing*)** | No presenta | **Sí** (marca la posición exacta del centro del borde) |
| **Sensibilidad al Ruido** | Alta | **Extrema** (amplifica drásticamente el error) |
| **Operador 2D Canónico** | **Gradiente** (Sobel, Prewitt) | **Laplaciano** ($\nabla^2$) |
| **Propósito Principal** | Detección de contornos gruesos y dirección | Realce de detalles finos (*unsharp masking*) |

---

## 📐 Operadores 2D en Imágenes

### 1. Gradiente (Vector 2D de Primera Derivada)
$$ \nabla f = \begin{bmatrix} G_x \\ G_y \end{bmatrix} = \begin{bmatrix} \frac{\partial f}{\partial x} \\ \frac{\partial f}{\partial y} \end{bmatrix}, \quad M(x, y) = \sqrt{G_x^2 + G_y^2} $$

### 2. Laplaciano (Escalar 2D de Segunda Derivada)
$$ \nabla^2 f = \frac{\partial^2 f}{\partial x^2} + \frac{\partial^2 f}{\partial y^2} $$
Máscaras estándar $3 \times 3$:
$$ K_4 = \begin{bmatrix} 0 & 1 & 0 \\ 1 & -4 & 1 \\ 0 & 1 & 0 \end{bmatrix}, \qquad K_8 = \begin{bmatrix} 1 & 1 & 1 \\ 1 & -8 & 1 \\ 1 & 1 & 1 \end{bmatrix} $$

---

## 🔗 Notas relacionadas

- 🧠 Conceptos:
  - [[Derivadas Espaciales en Imágenes]]
  - [[Filtrado Espacial y Convolución]]
  - [[Filtros de Suavizado Espacial]]
  - [[Manejo de Bordes y Padding en Imágenes]]
- 💻 Código:
  - [[Derivadas y Bordes - Python]]
- 📅 Sesión previa:
  - [[2026-09-23 - Filtros de Suavizado y Separabilidad de Kernels]]

## 📌 Pendientes / tareas

- [ ] Implementar el operador Laplaciano con `cv2.Laplacian` y observar la amplificación del ruido.
- [ ] Aplicar filtro Gaussiano previo antes de derivar para implementar el detector Laplaciano del Gaussiano (LoG).

## 🏷️ Etiquetas

#visión-artificial #clase #derivadas-espaciales #detección-de-bordes #gradiente #laplaciano #sobel #opencv #python

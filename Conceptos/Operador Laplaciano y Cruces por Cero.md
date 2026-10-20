---
titulo: "Operador Laplaciano y Cruces por Cero"
materia: Visión Artificial
tipo: concepto
fecha: 2026-10-20
tags:
  - visión-artificial
  - procesamiento-de-imágenes
  - laplaciano
  - segunda-derivada
  - zero-crossings
  - detección-de-bordes
aliases:
  - Laplaciano 2D
  - Cruces por cero
  - Zero-crossings
  - Segunda derivada espacial
---

# Operador Laplaciano y Cruces por Cero

## 🎯 Definición

El **Operador Laplaciano ($\nabla^2 f$)** es el operador diferencial lineal de segundo orden e **isotrópico** (invariante a rotaciones) por excelencia en procesamiento digital de imágenes. Mide la **tasa de cambio de la pendiente (curvatura espacial)** de la función de intensidad $f(x, y)$.

A diferencia del vector Gradiente ($\nabla f = [G_x, G_y]^T$), el Laplaciano entrega un **único valor escalar** por píxel:

$$ \nabla^2 f = \frac{\partial^2 f}{\partial x^2} + \frac{\partial^2 f}{\partial y^2} $$

---

## 🧮 Formulación Discreta y Máscaras Convolucionales

Aproximando las segundas derivadas parciales mediante diferencias finitas centrales con $\Delta x = \Delta y = 1$:

$$ \frac{\partial^2 f}{\partial x^2} \approx f(x+1, y) - 2f(x, y) + f(x-1, y) $$
$$ \frac{\partial^2 f}{\partial y^2} \approx f(x, y+1) - 2f(x, y) + f(x, y-1) $$

Sumando ambas componentes obtenemos la formulación canónica:

$$ \nabla^2 f(x, y) = f(x+1, y) + f(x-1, y) + f(x, y+1) + f(x, y-1) - 4f(x, y) $$

### Kernels $3 \times 3$ Estándar

```text
    Vecindad-4 (Cruz)               Vecindad-8 (Con Diagonales)
      [  0   1   0 ]                     [  1   1   1 ]
  K = [  1  -4   1 ]                 K = [  1  -8   1 ]
      [  0   1   0 ]                     [  1   1   1 ]
```

*(También es frecuente encontrar sus versiones con signos invertidos, donde el centro es positivo $+4$ u $+8$ y los vecinos son $-1$).*

> [!important] Condición de Suma Nula
> En todo kernel Laplaciano válido, **la suma algebraica de sus coeficientes debe ser idénticamente cero**:
> $$ \sum_{i} w_i = 0 $$
> Esto garantiza que en áreas homogéneas (zonas planas de brillo constante), la respuesta del filtro sea exactamente **$0$**.

---

## 🔬 Las Propiedades Fundamentales

### 1. Isotropía (Invarianza Rotacional)
Al ser una suma de derivadas segundas en ejes ortogonales, el Laplaciano es un operador escalar isotrópico. Responde con la misma magnitud a un borde sin importar su inclinación angular ($0^\circ, 45^\circ, 90^\circ$ o cualquier ángulo intermedio), eliminando la necesidad de calcular direcciones trigonométricas.

### 2. Detección por Cruces por Cero (*Zero-Crossings*)
El Laplaciano no localiza los bordes en los máximos de amplitud (como el gradiente), sino en el **cambio de signo**:
* Al atravesar una transición de oscuro a claro (rampa o escalón), la segunda derivada genera un **lóbulo positivo** en el lado inferior de la transición y un **lóbulo negativo** en el lado superior.
* El borde verdadero se sitúa exactamente en el punto de inflexión donde la curva cruza el valor **cero** ($+ \leftrightarrow -$).
* **Gran ventaja:** Los bordes detectados por cruces por cero tienen un grosor perfecto de **1 píxel** y forman contornos cerrados continuos.

```text
Intensidad:      Oscuro ---------/--------- Claro (Rampa)
1.ª Derivada:             ---/¨¨¨¨¨¨\---         (Joroba ancha)
2.ª Derivada:             ---+       -0-         (Cruce por cero)
                             |        |
                             \---+----/
```

### 3. Respuesta Doble (*Double Response*)
Para cada discontinuidad en la imagen, el Laplaciano produce dos líneas adyacentes: una clara en el borde interior y una oscura en el borde exterior.

### 4. Sensibilidad Extrema al Ruido
Al ser un operador de segundo orden:
* Multiplica cualquier píxel anómalo por $-4$ u $-8$.
* Su respuesta en el dominio de la frecuencia crece de forma **cuadrática ($\omega^2$)**, amplificando drásticamente el grano y el ruido de alta frecuencia. Por esta razón, **nunca debe aplicarse directamente sobre imágenes ruidosas** sin un suavizado Gaussiano previo.

---

## 🔗 Relacionado

- [[Filtro Laplaciano del Gaussiano - LoG]]
- [[Derivadas Espaciales en Imágenes]]
- [[Operadores de Gradiente - Roberts, Prewitt y Sobel]]
- [[Operador Laplaciano y Filtro LoG - Python]]

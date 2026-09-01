---
titulo: "Operaciones Aritméticas y Normalización - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-09-01
clase: "[[2026-09-01 - Operaciones Aritméticas y Lógicas entre Imágenes]]"
tags:
  - visión-artificial
  - python
  - opencv
  - scikit-image
  - matplotlib
  - operaciones-aritméticas
  - operaciones-lógicas
aliases:
  - Código suma y normalización de imágenes
  - Operaciones punto a punto OpenCV
  - Operaciones bitwise OpenCV
---

# Operaciones Aritméticas, Lógicas y Normalización — Python

> [!info] ¿Qué es este código?
> Este script completo cubre las operaciones punto a punto entre imágenes digitales con **OpenCV**, **NumPy** y **scikit-image**:
> 1. **Suma:** Desbordamiento modular (overflow), saturación y mezcla ponderada normalizada.
> 2. **Resta:** Resta saturada (`cv2.subtract`) frente a diferencia absoluta (`cv2.absdiff`).
> 3. **Máscaras sintéticas:** Dibujo de círculos y rectángulos con OpenCV.
> 4. **Álgebra de Boole:** Operadores a nivel de bits (`bitwise_and`, `bitwise_or`, `bitwise_xor`, `bitwise_not`).

> [!info] Clase y teoría relacionadas
> 📅 Clase: [[2026-09-01 - Operaciones Aritméticas y Lógicas entre Imágenes]]
> 🧠 Conceptos: [[Operaciones Aritméticas entre Imágenes]], [[Desbordamiento y Normalización de Imágenes]] y [[Operaciones Lógicas y Álgebra Booleana en Imágenes]]

---

## 📦 Requisitos

```bash
pip install opencv-python-headless scikit-image numpy matplotlib
```

---

## 💻 Código completo

```python
import numpy as np
import cv2
import matplotlib.pyplot as plt
from skimage import data

# Configuración global de tamaño de figuras
plt.rcParams["figure.figsize"] = (10, 5)

# ==============================================================================
# 1. CARGA Y HOMOGENEIZACIÓN DE DIMENSIONES
# ==============================================================================
img1 = data.camera()
img2 = data.coins()

# Ajustar img2 para que coincida exactamente con las dimensiones de img1 (w, h)
img2 = cv2.resize(img2, (img1.shape[1], img1.shape[0]))

# Visualización de imágenes fuente
fig, axis = plt.subplots(1, 2)
axis[0].imshow(img1, cmap="gray")
axis[0].set_title("Image 1 (Cameraman)")
axis[0].axis("off")

axis[1].imshow(img2, cmap="gray")
axis[1].set_title("Image 2 (Coins)")
axis[1].axis("off")
plt.tight_layout()
plt.show()


# ==============================================================================
# 2. OPERACIONES DE SUMA Y MANEJO DEL RANGO DINÁMICO
# ==============================================================================

# A. Desbordamiento Modular (Overflow en uint8 con NumPy):
# 200 + 100 = 300 -> 300 % 256 = 44 (Píxeles blancos se vuelven oscuros)
overflow_add = img1 + img2 

# B. Suma Saturada (OpenCV):
# Trunca cualquier valor > 255 al máximo 255 (Zona quemada)
saturated_add = cv2.add(img1, img2)

# C. Mezcla Ponderada / Normalizada:
# 0.5 * img1 + 0.5 * img2 + 0 -> Mantiene el rango sin quemar
normaliced_add = cv2.addWeighted(img1, 0.5, img2, 0.5, 0)

fig, axis = plt.subplots(1, 3, figsize=(15, 5))
axis[0].imshow(overflow_add, cmap="gray")
axis[0].set_title("Overflow (NumPy +)")
axis[0].axis("off")

axis[1].imshow(saturated_add, cmap="gray")
axis[1].set_title("Saturated (cv2.add)")
axis[1].axis("off")

axis[2].imshow(normaliced_add, cmap="gray")
axis[2].set_title("Normalized (cv2.addWeighted)")
axis[2].axis("off")
plt.tight_layout()
plt.show()


# ==============================================================================
# 3. OPERACIONES DE RESTA: SATURADA VS DIFERENCIA ABSOLUTA
# ==============================================================================

# A. Resta saturada (valores negativos se sustituyen por 0):
saturated_sub = cv2.subtract(img1, img2)

# B. Diferencia absoluta |img1 - img2| (detecta variaciones reales):
absolute_sub = cv2.absdiff(img1, img2)

fig, axis = plt.subplots(1, 2, figsize=(12, 5))
axis[0].imshow(saturated_sub, cmap="gray")
axis[0].set_title("Saturated Sub (cv2.subtract)")
axis[0].axis("off")

axis[1].imshow(absolute_sub, cmap="gray")
axis[1].set_title("Absolute Sub (cv2.absdiff)")
axis[1].axis("off")
plt.tight_layout()
plt.show()


# ==============================================================================
# 4. GENERACIÓN DE MÁSCARAS SINTÉTICAS
# ==============================================================================
h, w = img1.shape

# A. Máscara Circular (centrada, radio = min(h,w)//4)
msk_circle = np.zeros((h, w), dtype=np.uint8)
cv2.circle(
    msk_circle,
    (w // 2, h // 2),        # Coordenadas del centro (x, y)
    min(h, w) // 4,          # Radio del círculo
    255,                     # Color blanco
    -1                       # Grosor -1 = relleno sólido
)

# B. Máscara Rectangular (centrada entre w/3 y 2w/3)
msk_rect = np.zeros((h, w), dtype=np.uint8)
cv2.rectangle(
    msk_rect,
    (w // 3, h // 3),        # Esquina superior izquierda (x1, y1)
    (2 * w // 3, 2 * h // 3),# Esquina inferior derecha (x2, y2)
    255,                     # Color blanco
    -1                       # Relleno sólido
)

fig, axis = plt.subplots(1, 2, figsize=(10, 5))
axis[0].imshow(msk_circle, cmap="gray")
axis[0].set_title("Mask Circle")
axis[0].axis("off")

axis[1].imshow(msk_rect, cmap="gray")
axis[1].set_title("Mask Rectangle")
axis[1].axis("off")
plt.tight_layout()
plt.show()


# ==============================================================================
# 5. OPERACIONES LÓGICAS BIT A BIT (ÁLGEBRA DE BOOLE)
# ==============================================================================

# AND: Enmascaramiento para extraer la ROI circular sobre el cameraman
and_operator = cv2.bitwise_and(img1, img1, mask=msk_circle)

# OR: Unión de las dos máscaras geométricas
or_operator = cv2.bitwise_or(msk_circle, msk_rect)

# XOR: Diferencia simétrica (apaga la intersección y deja solo los bordes)
xor_operator = cv2.bitwise_xor(msk_circle, msk_rect)

# NOT: Inversión de la máscara circular (negativo)
not_operator = cv2.bitwise_not(msk_circle)

fig, axis = plt.subplots(1, 4, figsize=(18, 5))
axis[0].imshow(and_operator, cmap="gray")
axis[0].set_title("AND Operator (ROI)")
axis[0].axis("off")

axis[1].imshow(or_operator, cmap="gray")
axis[1].set_title("OR Operator (Unión)")
axis[1].axis("off")

axis[2].imshow(xor_operator, cmap="gray")
axis[2].set_title("XOR Operator (Diff)")
axis[2].axis("off")

axis[3].imshow(not_operator, cmap="gray")
axis[3].set_title("NOT Operator (Invert)")
axis[3].axis("off")

plt.tight_layout()
plt.show()
```

---

## 🔍 Explicación paso a paso de los resultados

### 1. Resta Saturada frente a Diferencia Absoluta
![[operaciones-resta-comparacion.png]]

- `cv2.subtract(img1, img2)`: Donde las monedas eran más brillantes que el fotógrafo, la resta dio números negativos truncados a **0 (negro)**, perdiéndose la forma de la persona en esas zonas.
- `cv2.absdiff(img1, img2)`: Al aplicar valor absoluto $|f_1 - f_2|$, preserva la silueta completa sin importar qué objeto sea más brillante.

### 2. Máscaras Sintéticas
![[operaciones-mascaras-sinteticas.png]]

- Permiten definir zonas espaciales arbitrarias en blanco ($255$) sobre fondo negro ($0$).

### 3. Álgebra de Boole sobre Imágenes
![[operaciones-logicas-bitwise.png]]

- **AND**: Aísla al cameraman dentro del círculo y apaga todo el exterior.
- **OR**: Combina el círculo y el rectángulo formando una sola región activa continua.
- **XOR**: Cancela la zona central donde se intersectan ambas formas, dejando únicamente los "arcos" exteriores.
- **NOT**: Invierte la máscara dejando el centro en negro y el exterior en blanco.

---

## 🔗 Relacionado

- [[Operaciones Aritméticas entre Imágenes]] — teoría
- [[Desbordamiento y Normalización de Imágenes]] — overflow y saturación
- [[Operaciones Lógicas y Álgebra Booleana en Imágenes]] — álgebra binaria
- [[2026-09-01 - Operaciones Aritméticas y Lógicas entre Imágenes]] — clase teórica
- [[Inicio]] — mapa de contenidos

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #scikit-image #matplotlib #operaciones-aritméticas #operaciones-lógicas #normalización

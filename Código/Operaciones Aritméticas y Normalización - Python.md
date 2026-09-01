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
aliases:
  - Código suma y normalización de imágenes
  - Operaciones punto a punto OpenCV
---

# Operaciones Aritméticas y Normalización — Python

> [!info] ¿Qué es este código?
> Este script demuestra cómo realizar operaciones aritméticas entre imágenes digitales con **OpenCV**, **NumPy** y **scikit-image**, comparando visualmente el efecto del **desbordamiento modular (overflow)**, la **saturación (clipping)** y la **normalización / mezcla ponderada**.

> [!info] Clase y teoría relacionadas
> 📅 Clase: [[2026-09-01 - Operaciones Aritméticas y Lógicas entre Imágenes]]
> 🧠 Conceptos: [[Operaciones Aritméticas entre Imágenes]] y [[Desbordamiento y Normalización de Imágenes]]

---

## 🎯 Qué hace el programa

1. Carga dos imágenes de prueba estándar (`cameraman` y `coins`) desde el paquete `skimage.data`.
2. Ajusta las dimensiones de la segunda imagen para que coincida exactamente con la primera usando `cv2.resize`.
3. Aplica **3 variantes de suma**:
   - **Desbordamiento Modular (`img1 + img2`)**: Aritmética `uint8` estándar en NumPy con módulo 256.
   - **Suma Saturada (`cv2.add(img1, img2)`)**: Trunca cualquier valor $> 255$ fijándolo en 255.
   - **Mezcla Ponderada / Normalizada (`cv2.addWeighted`)**: Aplica $\alpha f_1 + \beta f_2$ para mantener las intensidades dentro del rango visible sin quemar la imagen.
4. Grafica y contrasta los resultados en subplots de Matplotlib.

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

# Configuración de tamaño de figuras
plt.rcParams["figure.figsize"] = (10, 5)

# 1. Carga de imágenes de muestra
img1 = data.camera()
img2 = data.coins()

# 2. Homogeneización de dimensiones (Requisito indispensable para operar matrices)
# Nota: img1.shape[1] = ancho (columnas), img1.shape[0] = alto (filas)
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
# 3. OPERACIONES DE SUMA Y MANEJO DEL RANGO DINÁMICO
# ==============================================================================

# A. Desbordamiento Modular (Overflow en uint8):
# Si 200 + 100 = 300 -> 300 % 256 = 44 (Pérdida drástica de brillo)
overflow_add = img1 + img2 

# B. Suma con Saturación / Truncamiento (OpenCV):
# Si el resultado > 255, se establece en 255 (Zona blanca/quemada)
saturated_add = cv2.add(img1, img2)

# C. Mezcla Ponderada (AddWeighted - Normalización al 50%):
# Formula: 0.5 * img1 + 0.5 * img2 + 0
normaliced_add = cv2.addWeighted(img1, 0.5, img2, 0.5, 0)

# D. Alternativa: Normalización Min-Max pura por software
suma_float = img1.astype(float) + img2.astype(float)
min_val, max_val = suma_float.min(), suma_float.max()
minmax_norm = 255.0 * ((suma_float - min_val) / (max_val - min_val))
minmax_norm = minmax_norm.astype(np.uint8)

# 4. Comparación de los resultados
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
```

---

## 🔍 Explicación paso a paso

### 1. Imágenes Fuente
![[operaciones-imagenes-fuente.png]]

Antes de ejecutar cualquier suma o resta elemento a elemento, ambas imágenes deben poseer exactamente el mismo tamaño espacial $(H \times W)$.

### 2. Comparación de Métodos de Suma
![[operaciones-suma-comparacion.png]]

- **Overflow (`img1 + img2`)**: Las monedas sobre el cielo blanco producen manchas oscuras inesperadas porque $250 + 80 = 330 \to 74$.
- **Saturated (`cv2.add`)**: El cielo y las monedas claras se saturan a $255$ (blanco plano), perdiendo textura interior.
- **Normalized (`cv2.addWeighted` / Min-Max)**: Ambas imágenes se fusionan en transparencia perfecta sin quemar las altas luces ni introducir artefactos oscuros.

---

## ⚠️ Errores comunes

1. **Intentar operar imágenes con distinta resolución**:
   - `ValueError: operands could not be broadcast together with shapes...`
   - **Solución:** Redimensionar con `cv2.resize(img2, (img1.shape[1], img1.shape[0]))`.
2. **Confundir la suma de NumPy con la de OpenCV**:
   - `img1 + img2` $\to$ Aritmética modular `uint8`.
   - `cv2.add(img1, img2)` $\to$ Saturación a 255.
3. **División por cero en corrección de sombreado**:
   - Al dividir matrices, sumar un pequeño epsilon ($\epsilon = 10^{-6}$) al denominador para evitar divisiones indeterminadas.

---

## 🔗 Relacionado

- [[Operaciones Aritméticas entre Imágenes]] — teoría de suma, resta, multiplicación y división
- [[Desbordamiento y Normalización de Imágenes]] — formulación matemática
- [[2026-09-01 - Operaciones Aritméticas y Lógicas entre Imágenes]] — clase teórica
- [[Inicio]] — mapa de contenidos

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #scikit-image #matplotlib #operaciones-aritméticas #normalización

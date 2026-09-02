---
titulo: "Transformaciones de Intensidad y Histogramas - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-09-02
clase: "[[2026-09-02 - Transformaciones de Intensidad y Procesamiento de Histogramas]]"
tags:
  - visión-artificial
  - python
  - opencv
  - matplotlib
  - histogramas
  - ecualización
  - corrección-gamma
aliases:
  - Código transformaciones de intensidad
  - Código ecualización de histograma OpenCV
---

# Transformaciones de Intensidad y Histogramas — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python (OpenCV, NumPy y Matplotlib)** las transformaciones de intensidad en el dominio espacial más importantes: **Negativo**, **Logarítmica**, **Corrección Gamma**, **Estiramiento de Contraste** y **Ecualización de Histograma**.

> [!info] Clase y conceptos relacionados
> 📅 Clase: [[2026-09-02 - Transformaciones de Intensidad y Procesamiento de Histogramas]]
> 🧠 Conceptos: [[Transformaciones de Intensidad Espacial]] y [[Histogramas y Ecualización de Imagen]]

---

## 📦 Requisitos

```bash
pip install opencv-python numpy matplotlib scikit-image
```

---

## 💻 Código completo

```python
import numpy as np
import cv2
import matplotlib.pyplot as plt
from skimage import data

# ==============================================================================
# 1. CARGA DE IMAGEN DE PRUEBA
# ==============================================================================
img = data.camera()  # Imagen en escala de grises (uint8, 0-255)

# ==============================================================================
# 2. TRANSFORMACIONES DE INTENSIDAD BÁSICAS
# ==============================================================================

# A. Inversión Fotográfica / Negativo: s = 255 - r
img_negativo = 255 - img

# B. Transformación Logarítmica: s = c * log(1 + r)
# c = 255 / log(1 + max_pixel)
c_log = 255.0 / np.log(1.0 + np.max(img))
img_log = c_log * np.log(1.0 + img.astype(float))
img_log = np.array(img_log, dtype=np.uint8)

# C. Corrección Gamma (Ley de Potencias): s = 255 * (r / 255)^gamma
gamma_low = 0.4   # Expande oscuros (aclara)
gamma_high = 2.2  # Comprime oscuros (oscurece)

img_gamma_low = np.array(255 * (img / 255.0) ** gamma_low, dtype=np.uint8)
img_gamma_high = np.array(255 * (img / 255.0) ** gamma_high, dtype=np.uint8)

# D. Estiramiento de Contraste por Tramos (Min-Max)
r_min, r_max = float(np.min(img)), float(np.max(img))
img_stretched = 255.0 * ((img.astype(float) - r_min) / (r_max - r_min))
img_stretched = img_stretched.astype(np.uint8)

# ==============================================================================
# 3. ECUALIZACIÓN DE HISTOGRAMA CON OPENCV
# ==============================================================================
img_eq = cv2.equalizeHist(img)

# ==============================================================================
# 4. VISUALIZACIÓN COMPARATIVA DE TRANSFORMACIONES
# ==============================================================================
fig, axes = plt.subplots(2, 3, figsize=(16, 10))

axes[0, 0].imshow(img, cmap='gray')
axes[0, 0].set_title('Original')
axes[0, 0].axis('off')

axes[0, 1].imshow(img_negativo, cmap='gray')
axes[0, 1].set_title('Negativo (255 - r)')
axes[0, 1].axis('off')

axes[0, 2].imshow(img_log, cmap='gray')
axes[0, 2].set_title('Logarítmica (c*log(1+r))')
axes[0, 2].axis('off')

axes[1, 0].imshow(img_gamma_low, cmap='gray')
axes[1, 0].set_title(f'Gamma aclarado (γ = {gamma_low})')
axes[1, 0].axis('off')

axes[1, 1].imshow(img_gamma_high, cmap='gray')
axes[1, 1].set_title(f'Gamma oscurecido (γ = {gamma_high})')
axes[1, 1].axis('off')

axes[1, 2].imshow(img_eq, cmap='gray')
axes[1, 2].set_title('Ecualización de Histograma')
axes[1, 2].axis('off')

plt.tight_layout()
plt.show()

# ==============================================================================
# 5. COMPARACIÓN DE HISTOGRAMAS: ORIGINAL VS ECUALIZADA
# ==============================================================================
fig, axes = plt.subplots(2, 2, figsize=(14, 8))

# Imagen original y su histograma
axes[0, 0].imshow(img, cmap='gray')
axes[0, 0].set_title('Imagen Original')
axes[0, 0].axis('off')

axes[0, 1].hist(img.ravel(), bins=256, range=[0, 256], color='black', alpha=0.7)
axes[0, 1].set_title('Histograma Original')
axes[0, 1].set_xlim([0, 256])

# Imagen ecualizada y su histograma
axes[1, 0].imshow(img_eq, cmap='gray')
axes[1, 0].set_title('Imagen Ecualizada')
axes[1, 0].axis('off')

axes[1, 1].hist(img_eq.ravel(), bins=256, range=[0, 256], color='royalblue', alpha=0.7)
axes[1, 1].set_title('Histograma Ecualizado (Plano / Expandido)')
axes[1, 1].set_xlim([0, 256])

plt.tight_layout()
plt.show()
```

---

## 🔍 Explicación paso a paso

1. **`img.ravel()`**:
   - Aplana la matriz bidimensional $M \times N$ a un vector unidimensional $1\text{D}$, necesario para que `plt.hist()` contabilice las frecuencias de cada intensidad rápidamente.
2. **`cv2.equalizeHist(src)`**:
   - Calcula internamente la función de distribución acumulada (CDF) y mapea los niveles de gris para obtener un contraste máximo. Solo admite imágenes de un solo canal en `uint8`.
3. **Conversión a coma flotante para operaciones no lineales**:
   - Para $\log(1+r)$ o $(r/255)^\gamma$, es obligatorio convertir a `float` antes de operar, y al finalizar reconvertir a `np.uint8`.

---

## ⚠️ Errores comunes

- **Olvidar normalizar a $[0, 1]$ antes de aplicar Gamma**:
  - Si calculas `img ** gamma` directamente con valores entre $0$ y $255$, $255^{2.2} \approx 198\,000$, desbordando la memoria.
  - **Solución correcta:** `255 * (img / 255.0) ** gamma`.
- **Aplicar `cv2.equalizeHist` a imágenes RGB directamente**:
  - Genera aberraciones cromáticas. Para imágenes a color se debe convertir al espacio **HSV** o **YCrCb** y ecualizar únicamente el canal de luminancia ($V$ o $Y$).

---

## 🔗 Relacionado

- [[Transformaciones de Intensidad Espacial]] — teoría matemática
- [[Histogramas y Ecualización de Imagen]] — histogramas y CDF
- [[2026-09-02 - Transformaciones de Intensidad y Procesamiento de Histogramas]] — clase teórica
- [[Inicio]] — mapa general de la materia

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #matplotlib #histogramas #ecualización #corrección-gamma

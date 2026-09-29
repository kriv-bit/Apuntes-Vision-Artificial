---
titulo: "Derivadas y Detección de Bordes - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-09-29
clase: "[[2026-09-29 - Derivadas Espaciales y Detección de Bordes]]"
tags:
  - visión-artificial
  - python
  - opencv
  - matplotlib
  - derivadas-espaciales
  - detección-de-bordes
  - gradiente
  - laplaciano
aliases:
  - Código derivadas espaciales
  - Código Sobel y Laplaciano
  - cv2.Sobel y cv2.Laplacian
---

# Derivadas y Detección de Bordes — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python y OpenCV**:
> 1. Verificación numérica 1D del vector de clase con diferencias finitas de primer y segundo orden.
> 2. **Primera derivada 2D (Gradiente de Sobel)** en $X$, $Y$ y cálculo de su magnitud.
> 3. **Segunda derivada 2D (Operador Laplaciano)** con `cv2.Laplacian`.
> 4. Demostración práctica de la **extrema sensibilidad al ruido** del Laplaciano y su solución mediante pre-suavizado Gaussiano (**Laplaciano del Gaussiano - LoG**).

> [!info] Clases y conceptos relacionados
> 📅 Clase: [[2026-09-29 - Derivadas Espaciales y Detección de Bordes]]
> 🧠 Concepto: [[Derivadas Espaciales en Imágenes]] y [[Filtros de Suavizado Espacial]]

---

## 📦 Requisitos

```bash
pip install opencv-python-headless numpy matplotlib scikit-image
```

---

## 💻 Código completo

```python
import numpy as np
import cv2
import matplotlib.pyplot as plt
from skimage import data

# Configuración global de figuras
plt.rcParams["figure.figsize"] = (12, 6)

# ==============================================================================
# 1. VERIFICACIÓN NUMÉRICA 1D (EJEMPLO DE CLASE)
# ==============================================================================
# Vector de intensidades con rampa ascendente
p = np.array([30, 30, 30, 100, 150, 150, 150], dtype=np.int32)

# Primera derivada: d[i] = p[i+1] - p[i]
d1 = np.diff(p)

# Segunda derivada: d2[i] = d1[i+1] - d1[i]
d2 = np.diff(d1)

print("=== VERIFICACIÓN NUMÉRICA 1D ===")
print("Vector original p       :", p)
print("Primera Derivada d1     :", d1)
print("Segunda Derivada d2     :", d2)
# d1 esperada: [ 0  0 70 50  0  0]
# d2 esperada: [  0  70 -20 -50   0]


# ==============================================================================
# 2. PRIMERA DERIVADA 2D: OPERADOR DE SOBEL (GRADIENTE)
# ==============================================================================
img = data.camera()

# cv2.CV_64F es obligatorio para registrar derivadas negativas
sobel_x = cv2.Sobel(img, cv2.CV_64F, dx=1, dy=0, ksize=3)
sobel_y = cv2.Sobel(img, cv2.CV_64F, dx=0, dy=1, ksize=3)

# Magnitud del gradiente: sqrt(Gx^2 + Gy^2)
sobel_magnitude = cv2.magnitude(sobel_x, sobel_y)

# Normalizar para visualización en 8 bits [0, 255]
sobel_mag_uint8 = np.uint8(np.clip(sobel_magnitude, 0, 255))


# ==============================================================================
# 3. SEGUNDA DERIVADA 2D: OPERADOR LAPLACIANO
# ==============================================================================
laplacian = cv2.Laplacian(img, cv2.CV_64F, ksize=3)
laplacian_abs = np.uint8(np.clip(np.abs(laplacian), 0, 255))


# ==============================================================================
# 4. SENSIBILIDAD AL RUIDO Y SOLUCIÓN LoG (LAPLACIAN OF GAUSSIAN)
# ==============================================================================
# Agregar ruido gaussiano aditivo a la imagen
noise = np.random.normal(0, 15, img.shape)
noisy_img = np.clip(img.astype(float) + noise, 0, 255).astype(np.uint8)

# Laplaciano aplicado DIRECTAMENTE sobre la imagen ruidosa (Catástrofe)
laplacian_ruido = cv2.Laplacian(noisy_img, cv2.CV_64F, ksize=3)
laplacian_ruido_abs = np.uint8(np.clip(np.abs(laplacian_ruido), 0, 255))

# Solución LoG: Suavizado Gaussiano previo + Laplaciano
blurred_img = cv2.GaussianBlur(noisy_img, (5, 5), sigmaX=1.5)
log_result = cv2.Laplacian(blurred_img, cv2.CV_64F, ksize=3)
log_abs = np.uint8(np.clip(np.abs(log_result), 0, 255))


# ==============================================================================
# 5. VISUALIZACIÓN COMPARATIVA
# ==============================================================================
fig, axes = plt.subplots(2, 3, figsize=(16, 10))

# Fila 1: Imagen limpia y sus derivadas
axes[0, 0].imshow(img, cmap='gray')
axes[0, 0].set_title('Original Limpia')
axes[0, 0].axis('off')

axes[0, 1].imshow(sobel_mag_uint8, cmap='gray')
axes[0, 1].set_title('1ra Derivada: Magnitud Sobel (Bordes Gruesos)')
axes[0, 1].axis('off')

axes[0, 2].imshow(laplacian_abs, cmap='gray')
axes[0, 2].set_title('2da Derivada: Laplaciano (Bordes Finos y Detalle)')
axes[0, 2].axis('off')

# Fila 2: Impacto del ruido y solución LoG
axes[1, 0].imshow(noisy_img, cmap='gray')
axes[1, 0].set_title('Imagen con Ruido Gaussiano')
axes[1, 0].axis('off')

axes[1, 1].imshow(laplacian_ruido_abs, cmap='gray')
axes[1, 1].set_title('Laplaciano Sin Filtro\n(El ruido domina por completo)')
axes[1, 1].axis('off')

axes[1, 2].imshow(log_abs, cmap='gray')
axes[1, 2].set_title('Laplaciano del Gaussiano (LoG)\n(Ruido atenuado, bordes recuperados)')
axes[1, 2].axis('off')

plt.tight_layout()
plt.show()
```

---

## 🔍 Puntos Críticos de Implementación

1. **Uso de `cv2.CV_64F` en vez de `uint8`:**
   - La derivada mide diferencias. Si un píxel pasa de $150$ a $30$, la derivada vale $-120$. Si calculas con `np.uint8`, el valor negativo provocará *underflow* modular y se convertirá erróneamente en $+136$.
2. **Extrema vulnerabilidad del Laplaciano:**
   - La comparación visual entre `Laplaciano Sin Filtro` y `Laplaciano del Gaussiano` demuestra en la práctica por qué la 2da derivada jamás debe usarse directamente sobre sensores reales sin un filtrado paso bajo previo.

---

## 🔗 Relacionado

- [[Derivadas Espaciales en Imágenes]] — teoría analítica
- [[Filtros de Suavizado Espacial]] — filtro Gaussiano previo
- [[2026-09-29 - Derivadas Espaciales y Detección de Bordes]] — clase teórica
- [[Inicio]] — mapa general de la materia

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #derivadas-espaciales #detección-de-bordes #gradiente #laplaciano #log

---
titulo: "Operador Laplaciano y Filtro LoG - Python"
materia: Visión Artificial
tipo: codigo
fecha: 2026-10-20
tags:
  - visión-artificial
  - opencv
  - python
  - laplaciano
  - log
  - zero-crossings
  - filtrado-espacial
aliases:
  - Laplaciano en Python
  - LoG en OpenCV
---

# Operador Laplaciano y Filtro LoG — Python

Implementación práctica en Python del operador Laplaciano discreto, detección de cruces por cero (*zero-crossings*) y el filtro Laplaciano del Gaussiano (LoG).

---

## 🐍 1. Laplaciano Discreto en OpenCV

OpenCV ofrece la función directa `cv2.Laplacian()` que opera en precisión de punto flotante `cv2.CV_64F`:

```python
import cv2
import numpy as np
import matplotlib.pyplot as plt
from skimage import data

# Cargar imagen en escala de grises
img = data.camera()
img_float = img.astype(np.float64)

# 1. Laplaciano directo (ksize=1 utiliza el kernel canónico 3x3 de vecindad-4)
lap_4 = cv2.Laplacian(img_float, cv2.CV_64F, ksize=1)

# 2. Laplaciano con kernel manual de 8-vecinos
kernel_8 = np.array([
    [1,  1,  1],
    [1, -8,  1],
    [1,  1,  1]
], dtype=np.float64)

lap_8 = cv2.filter2D(img_float, -1, kernel_8)

# Visualización
fig, axes = plt.subplots(1, 3, figsize=(15, 5))
axes[0].imshow(img, cmap='gray')
axes[0].set_title("Original")
axes[0].axis('off')

axes[1].imshow(np.abs(lap_4), cmap='gray')
axes[1].set_title("Laplaciano (4-vecinos)")
axes[1].axis('off')

axes[2].imshow(np.abs(lap_8), cmap='gray')
axes[2].set_title("Laplaciano (8-vecinos)")
axes[2].axis('off')

plt.tight_layout()
plt.show()
```

---

## 🐍 2. Implementación del Filtro LoG (Laplaciano del Gaussiano)

```python
def laplacian_of_gaussian(image, sigma=1.5):
    """
    Aplica el filtro LoG en dos etapas equivalentes:
    1. Suavizado Gaussiano con sigma
    2. Laplaciano sobre la imagen limpia
    """
    # Tamaño del kernel recomendado: 6*sigma + 1 (redondeado a impar)
    ksize = int(np.ceil(sigma * 6))
    if ksize % 2 == 0:
        ksize += 1
    
    # Paso 1: Suavizado Gaussiano
    blurred = cv2.GaussianBlur(image.astype(np.float64), (ksize, ksize), sigmaX=sigma, sigmaY=sigma)
    
    # Paso 2: Laplaciano
    log_response = cv2.Laplacian(blurred, cv2.CV_64F, ksize=1)
    return log_response

# Comparar diferentes escalas de sigma
log_s1 = laplacian_of_gaussian(img, sigma=1.0)
log_s2 = laplacian_of_gaussian(img, sigma=2.5)
log_s4 = laplacian_of_gaussian(img, sigma=4.0)

fig, axes = plt.subplots(1, 3, figsize=(16, 5))
axes[0].imshow(np.abs(log_s1), cmap='gray')
axes[0].set_title("LoG ($\sigma = 1.0$) - Detalles finos")
axes[0].axis('off')

axes[1].imshow(np.abs(log_s2), cmap='gray')
axes[1].set_title("LoG ($\sigma = 2.5$) - Escala intermedia")
axes[1].axis('off')

axes[2].imshow(np.abs(log_s4), cmap='gray')
axes[2].set_title("LoG ($\sigma = 4.0$) - Contornos globales")
axes[2].axis('off')

plt.tight_layout()
plt.show()
```

---

## 🐍 3. Detección de Cruces por Cero (*Zero-Crossings*)

```python
def find_zero_crossings(log_img, threshold=0.0):
    """
    Localiza cruces por cero en una respuesta LoG:
    Verifica si vecinos opuestos tienen signos distintos y la diferencia supera un umbral.
    """
    h, w = log_img.shape
    zc_map = np.zeros((h, w), dtype=np.uint8)
    
    # Desplazamientos para pares opuestos: (arriba-abajo, izq-der, diag1, diag2)
    pairs = [
        ((-1, 0), (1, 0)),   # Vertical
        ((0, -1), (0, 1)),   # Horizontal
        ((-1, -1), (1, 1)),  # Diagonal principal
        ((-1, 1), (1, -1))   # Diagonal secundaria
    ]
    
    for (dy1, dx1), (dy2, dx2) in pairs:
        # Desplazamientos matriciales con slicing
        p1 = log_img[1:-1, 1:-1]
        p_opp1 = log_img[1+dy1:h-1+dy1, 1+dx1:w-1+dx1]
        p_opp2 = log_img[1+dy2:h-1+dy2, 1+dx2:w-1+dx2]
        
        # Condición de cruce por cero entre vecinos opuestos
        cond1 = (p_opp1 * p_opp2 < 0) & (np.abs(p_opp1 - p_opp2) > threshold)
        zc_map[1:-1, 1:-1] |= cond1.astype(np.uint8) * 255
        
    return zc_map

# Detectar bordes finos de 1 píxel con cruces por cero
zc_borders = find_zero_crossings(log_s2, threshold=15.0)

plt.figure(figsize=(8, 8))
plt.imshow(zc_borders, cmap='gray')
plt.title("Bordes por Cruces por Cero (Zero-Crossings a 1 px)", fontweight='bold')
plt.axis('off')
plt.show()
```

---

## 🔗 Relacionado

- [[Operador Laplaciano y Cruces por Cero]]
- [[Filtro Laplaciano del Gaussiano - LoG]]
- [[2026-10-20 - Operador Laplaciano, Cruces por Cero y Filtro LoG]]
- [[Derivadas y Bordes - Python]]

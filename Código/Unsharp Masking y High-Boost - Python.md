---
titulo: "Unsharp Masking y High-Boost - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-09-30
clase: "[[2026-09-30 - Realce de Imágenes - Unsharp Masking y High-Boost Filtering]]"
tags:
  - visión-artificial
  - python
  - opencv
  - matplotlib
  - unsharp-masking
  - high-boost
  - realce-de-bordes
aliases:
  - Código Unsharp Masking
  - Código High-Boost Filtering
  - Realce de nitidez en OpenCV
---

# Unsharp Masking y High-Boost — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python y OpenCV**:
> 1. El algoritmo clásico de **Unsharp Masking** ($A = 1$) y **High-Boost Filtering** ($A > 1$).
> 2. Comparativa visual de nitidez variando el factor de amplificación $A$.
> 3. Demostración práctica del **costo oculto** (amplificación agresiva del ruido de sensor) y su solución mediante **Unsharp Masking con umbral** (*thresholding*).

> [!info] Clases y conceptos relacionados
> 📅 Clase: [[2026-09-30 - Realce de Imágenes: Unsharp Masking y High-Boost Filtering]]
> 🧠 Concepto: [[Unsharp Masking y High-Boost Filtering]] y [[Filtros de Suavizado Espacial]]

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

img = data.moon()  # Excelente imagen astronómica con cráteres y detalles finos

# ==============================================================================
# 1. FUNCIÓN MODULAR DE UNSHARP MASKING Y HIGH-BOOST FILTERING
# ==============================================================================
def unsharp_highboost(image, A=1.0, ksize=(5, 5), sigma=1.0, threshold=0.0):
    """
    Aplica Unsharp Masking (A=1) o High-Boost Filtering (A>1).
    - image: Imagen en escala de grises (uint8).
    - A: Factor de ponderación de la máscara.
    - ksize, sigma: Parámetros del filtro Gaussiano de desenfoque.
    - threshold: Umbral mínimo para aplicar realce (evita amplificar ruido plano).
    """
    img_f = image.astype(np.float64)
    
    # 1. Suavizado paso bajo con filtro Gaussiano
    blurred = cv2.GaussianBlur(img_f, ksize, sigmaX=sigma)
    
    # 2. Generación de la máscara de altas frecuencias
    mask = img_f - blurred
    
    # Aplicar umbral si se requiere mitigar ruido
    if threshold > 0.0:
        mask = np.where(np.abs(mask) >= threshold, mask, 0.0)
    
    # 3. Recombinación y clipping seguro en [0, 255]
    enhanced = img_f + (A * mask)
    return np.uint8(np.clip(enhanced, 0, 255)), mask


# ==============================================================================
# 2. COMPARATIVA DE REALCE VARIANDO EL FACTOR A
# ==============================================================================
out_um, mask_detail = unsharp_highboost(img, A=1.0, sigma=1.5)      # Unsharp estándar
out_hb_med, _ = unsharp_highboost(img, A=2.5, sigma=1.5)           # High-Boost moderado
out_hb_high, _ = unsharp_highboost(img, A=5.0, sigma=1.5)          # High-Boost agresivo

fig, axes = plt.subplots(1, 4, figsize=(18, 5))

axes[0].imshow(img, cmap='gray')
axes[0].set_title('Original')
axes[0].axis('off')

axes[1].imshow(out_um, cmap='gray')
axes[1].set_title('Unsharp Masking (A = 1.0)\nRealce Natural')
axes[1].axis('off')

axes[2].imshow(out_hb_med, cmap='gray')
axes[2].set_title('High-Boost (A = 2.5)\nBordes Acentuados')
axes[2].axis('off')

axes[3].imshow(out_hb_high, cmap='gray')
axes[3].set_title('High-Boost (A = 5.0)\nNitidez Extrema / Halos')
axes[3].axis('off')

plt.tight_layout()
plt.show()


# ==============================================================================
# 3. VISUALIZACIÓN DE LA MÁSCARA DE ALTAS FRECUENCIAS
# ==============================================================================
# Centrar la máscara en 128 para visualizar picos positivos (claros) y negativos (oscuros)
mask_viz = np.uint8(np.clip(mask_detail + 128.0, 0, 255))

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

axes[0].imshow(img, cmap='gray')
axes[0].set_title('Imagen de Entrada')
axes[0].axis('off')

axes[1].imshow(mask_viz, cmap='gray')
axes[1].set_title('Máscara g_mask = f - f_blur\n(Zonas planas colapsadas a gris 128)')
axes[1].axis('off')

plt.tight_layout()
plt.show()


# ==============================================================================
# 4. EL COSTO OCULTO: AMPLIFICACIÓN DE RUIDO Y SOLUCIÓN CON UMBRAL
# ==============================================================================
# Contaminar la imagen con ruido gaussiano leve
noise = np.random.normal(0, 8, img.shape)
noisy_img = np.uint8(np.clip(img.astype(float) + noise, 0, 255))

# High-Boost agresivo sin umbral (El grano explota)
hb_noisy, _ = unsharp_highboost(noisy_img, A=3.0, sigma=1.5, threshold=0.0)

# High-Boost con umbral T = 12.0 (Solo bordes reales se amplifican)
hb_filtered, _ = unsharp_highboost(noisy_img, A=3.0, sigma=1.5, threshold=12.0)

fig, axes = plt.subplots(1, 3, figsize=(16, 5))

axes[0].imshow(noisy_img, cmap='gray')
axes[0].set_title('Imagen con Ruido de Sensor')
axes[0].axis('off')

axes[1].imshow(hb_noisy, cmap='gray')
axes[1].set_title('High-Boost Sin Umbral (A = 3.0)\nCosto Oculto: Ruido Hiperamplificado')
axes[1].axis('off')

axes[2].imshow(hb_filtered, cmap='gray')
axes[2].set_title('High-Boost Con Umbral (T = 12.0)\nBordes Realzados sin amplificar ruido')
axes[2].axis('off')

plt.tight_layout()
plt.show()
```

---

## 🔍 Conclusiones de la Implementación

1. **El efecto de la acutancia:**  
   Al contrastar la imagen original de la Luna con $A = 2.5$, los cráteres y surcos del terreno lunar adquieren un relieve y separación visual impresionante gracias al sobreimpulso en los márgenes de los bordes.
2. **Importancia del `np.clip`:**  
   Al multiplicar por $A > 1$, las sumas sobrepasan frecuentemente $255$ o caen por debajo de $0$. El truncamiento seguro con `np.clip(..., 0, 255)` antes de convertir a `np.uint8` es estrictamente indispensable para evitar *overflow* cíclico.
3. **Control del costo oculto:**  
   El parámetro `threshold` filtra los microcambios generados por ruido estocástico, asegurando que la amplificación por $A$ solo se aplique a bordes verdaderos de alto contraste.

---

## 🔗 Relacionado

- [[Unsharp Masking y High-Boost Filtering]] — fundamentos teóricos
- [[Filtros de Suavizado Espacial]] — desenfoque Gaussiano
- [[2026-09-30 - Realce de Imágenes: Unsharp Masking y High-Boost Filtering]] — clase teórica
- [[Inicio]] — mapa general de la materia

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #unsharp-masking #high-boost #realce-de-bordes #nitidez

---
titulo: "Operadores de Gradiente - Roberts, Prewitt y Sobel - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-10-07
clase: "[[2026-10-07 - Operadores de Gradiente - Roberts, Prewitt y Sobel]]"
tags:
  - visión-artificial
  - python
  - opencv
  - matplotlib
  - detección-de-bordes
  - gradiente
  - sobel
  - prewitt
  - roberts
aliases:
  - Código Sobel Prewitt Roberts
  - Detección de bordes con gradiente en Python
---

# Operadores de Gradiente: Roberts, Prewitt y Sobel — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python y OpenCV**:
> 1. Los 3 operadores clásicos de primera derivada: **Roberts ($2 \times 2$)**, **Prewitt ($3 \times 3$)** y **Sobel ($3 \times 3$)**.
> 2. Comparativa visual de robustez ante **ruido Gaussiano**: demostración de por qué Roberts falla drásticamente y Sobel produce los bordes más limpios.
> 3. Cálculo de la **magnitud del gradiente** y **umbralización (*thresholding*)** para binarizar los bordes.

> [!info] Clases y conceptos relacionados
> 📅 Clase: [[2026-10-07 - Operadores de Gradiente - Roberts, Prewitt y Sobel]]
> 🧠 Conceptos: [[Operadores de Gradiente - Roberts, Prewitt y Sobel]] y [[Derivadas Espaciales en Imágenes]]

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
plt.rcParams["figure.figsize"] = (14, 8)

img = data.camera()

# ==============================================================================
# 1. DEFINICIÓN DE KERNELS DE GRADIENTE
# ==============================================================================
# Operador Cruzado de Roberts (2x2)
roberts_x = np.array([[1, 0], [0, -1]], dtype=np.float32)
roberts_y = np.array([[0, 1], [-1, 0]], dtype=np.float32)

# Operador de Prewitt (3x3)
prewitt_x = np.array([[-1, 0, 1], [-1, 0, 1], [-1, 0, 1]], dtype=np.float32)
prewitt_y = np.array([[-1, -1, -1], [0, 0, 0], [1, 1, 1]], dtype=np.float32)

# Operador de Sobel (3x3)
sobel_x_k = np.array([[-1, 0, 1], [-2, 0, 2], [-1, 0, 1]], dtype=np.float32)
sobel_y_k = np.array([[-1, -2, -1], [0, 0, 0], [1, 2, 1]], dtype=np.float32)

def compute_gradient_magnitude(image, kx, ky):
    """Calcula la magnitud del gradiente usando convolución 2D en float64"""
    gx = cv2.filter2D(image.astype(np.float64), -1, kx)
    gy = cv2.filter2D(image.astype(np.float64), -1, ky)
    mag = np.sqrt(gx**2 + gy**2)
    return mag


# ==============================================================================
# 2. COMPARATIVA EN IMAGEN LIMPIA
# ==============================================================================
mag_roberts = compute_gradient_magnitude(img, roberts_x, roberts_y)
mag_prewitt = compute_gradient_magnitude(img, prewitt_x, prewitt_y)
mag_sobel   = compute_gradient_magnitude(img, sobel_x_k, sobel_y_k)

# También se puede calcular Sobel nativo con OpenCV:
# gx_cv = cv2.Sobel(img, cv2.CV_64F, 1, 0, ksize=3)
# gy_cv = cv2.Sobel(img, cv2.CV_64F, 0, 1, ksize=3)
# mag_sobel = cv2.magnitude(gx_cv, gy_cv)

fig, axes = plt.subplots(1, 4, figsize=(16, 4))
axes[0].imshow(img, cmap='gray'); axes[0].set_title('Original'); axes[0].axis('off')
axes[1].imshow(mag_roberts, cmap='gray'); axes[1].set_title('Roberts (2x2)'); axes[1].axis('off')
axes[2].imshow(mag_prewitt, cmap='gray'); axes[2].set_title('Prewitt (3x3)'); axes[2].axis('off')
axes[3].imshow(mag_sobel, cmap='gray'); axes[3].set_title('Sobel (3x3)'); axes[3].axis('off')
plt.tight_layout()
plt.show()


# ==============================================================================
# 3. EL GRAN TEST: COMPARATIVA CON RUIDO DE SENSOR
# ==============================================================================
# Agregar ruido gaussiano
noise = np.random.normal(0, 15, img.shape)
noisy_img = np.uint8(np.clip(img.astype(float) + noise, 0, 255))

noisy_roberts = compute_gradient_magnitude(noisy_img, roberts_x, roberts_y)
noisy_prewitt = compute_gradient_magnitude(noisy_img, prewitt_x, prewitt_y)
noisy_sobel   = compute_gradient_magnitude(noisy_img, sobel_x_k, sobel_y_k)

fig, axes = plt.subplots(1, 4, figsize=(16, 4))
axes[0].imshow(noisy_img, cmap='gray')
axes[0].set_title('Imagen con Ruido')
axes[0].axis('off')

axes[1].imshow(noisy_roberts, cmap='gray')
axes[1].set_title('Roberts con Ruido\n(Destruido por ruido)')
axes[1].axis('off')

axes[2].imshow(noisy_prewitt, cmap='gray')
axes[2].set_title('Prewitt con Ruido\n(Atenuación media)')
axes[2].axis('off')

axes[3].imshow(noisy_sobel, cmap='gray')
axes[3].set_title('Sobel con Ruido\n(Bordes más limpios y robustos)')
axes[3].axis('off')

plt.tight_layout()
plt.show()


# ==============================================================================
# 4. UMBRALIZACIÓN DEL GRADIENTE (THRESHOLDING)
# ==============================================================================
# Binarizar el gradiente de Sobel según diferentes umbrales T
T_low = 60
T_mid = 120
T_high = 200

bin_low = np.where(mag_sobel >= T_low, 255, 0).astype(np.uint8)
bin_mid = np.where(mag_sobel >= T_mid, 255, 0).astype(np.uint8)
bin_high = np.where(mag_sobel >= T_high, 255, 0).astype(np.uint8)

fig, axes = plt.subplots(1, 3, figsize=(15, 5))
axes[0].imshow(bin_low, cmap='gray'); axes[0].set_title(f'Umbral Bajo (T = {T_low})\nMás bordes, algo de textura'); axes[0].axis('off')
axes[1].imshow(bin_mid, cmap='gray'); axes[1].set_title(f'Umbral Óptimo (T = {T_mid})\nBordes principales limpios'); axes[1].axis('off')
axes[2].imshow(bin_high, cmap='gray'); axes[2].set_title(f'Umbral Alto (T = {T_high})\nSolo bordes muy violentos'); axes[2].axis('off')
plt.tight_layout()
plt.show()
```

---

## 🔍 Conclusiones del Experimento

1. **El fracaso de Roberts:**
   - Al carecer de cualquier término de suavizado, cada mota de ruido se traduce en un gradiente gigante.
2. **Prewitt vs Sobel:**
   - Prewitt usa pesos iguales $[1, 1, 1]$, mientras que Sobel usa $[1, 2, 1]$. Esa simple ponderación central duplica la supresión de ruido aleatorio en bordes diagonales y horizontales.
3. **El compromiso del umbral ($T$):**
   - No existe un umbral mágico: umbrales bajos conservan detalles pero admiten falso grano; umbrales altos aíslan contornos pero desconectan líneas continuas (motivo por el cual John Canny inventó la umbralización por histéresis).

---

## 🔗 Relacionado

- [[Operadores de Gradiente - Roberts, Prewitt y Sobel]] — teoría analítica
- [[Derivadas Espaciales en Imágenes]] — fundamentos de gradientes
- [[2026-10-07 - Operadores de Gradiente - Roberts, Prewitt y Sobel]] — clase teórica
- [[Inicio]] — mapa general de la materia

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #detección-de-bordes #gradiente #sobel #prewitt #roberts

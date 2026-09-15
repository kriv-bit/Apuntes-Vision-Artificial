---
titulo: "Filtrado Espacial y Modos de Padding - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-09-09
clase: "[[2026-09-15 - Implementación de Convolución y Modos de Borde]]"
tags:
  - visión-artificial
  - python
  - opencv
  - scipy
  - matplotlib
  - filtrado-espacial
  - convolución
  - padding
aliases:
  - Código convolución vs correlación
  - Código modos de padding OpenCV
  - cv2.copyMakeBorder y cv2.filter2D
---

# Filtrado Espacial y Modos de Padding — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python (NumPy, OpenCV, SciPy y scikit-image)**:
> 1. La función manual de convolución rotando el kernel 180° con slicing de NumPy (`kernel[::-1, ::-1]`).
> 2. Comparativa experimental entre **Correlación** y **Convolución** usando **kernels simétricos** (idénticos) y **kernels asimétricos** (inversión de signos en gradientes).
> 3. Comparativa visual de los modos de **padding de bordes** (`BORDER_CONSTANT`, `BORDER_REPLICATE`, `BORDER_REFLECT`) sobre dos imágenes de prueba.

> [!info] Clases y conceptos relacionados
> 📅 Clases: [[2026-09-09 - Filtrado Espacial, Convolución y Manejo de Bordes]] y [[2026-09-15 - Implementación de Convolución y Modos de Borde]]
> 🧠 Conceptos: [[Filtrado Espacial y Convolución]] y [[Manejo de Bordes y Padding en Imágenes]]

---

## 📦 Requisitos

```bash
pip install opencv-python-headless numpy matplotlib scipy scikit-image
```

---

## 💻 Código completo

```python
import numpy as np
import cv2
import matplotlib.pyplot as plt
from scipy import ndimage
from skimage import data

# Configuración global de figuras
plt.rcParams["figure.figsize"] = (10, 10)

# Cargar imágenes de prueba estándar
img1 = data.camera()
img2 = data.coins()

# ==============================================================================
# 1. FUNCIÓN DE CONVOLUCIÓN MANUAL MEDIANTE ROTACIÓN DE 180°
# ==============================================================================
def correlation(image, kernel):
    """Correlación espacial 2D usando scipy.ndimage"""
    return ndimage.correlate(image, kernel, mode='reflect')

def convolution_manual(image, kernel):
    """
    Convolución espacial: igual que la correlación pero rotamos el kernel 180°.
    En NumPy, el slice [::-1, ::-1] invierte filas y columnas en O(1).
    """
    kernel_r = kernel[::-1, ::-1]
    return correlation(image, kernel_r)


# ==============================================================================
# 2. EXPERIMENTO A: KERNEL SIMÉTRICO (PROMEDIO / SUAVIZADO)
# ==============================================================================
# Kernel simétrico de promedio 3x3
kernel_p = np.ones((3, 3), dtype=np.float64) / 9.0

# Procesamiento de ambas imágenes
corr_s_img1 = correlation(img1, kernel_p)
conv_s_img1 = convolution_manual(img1, kernel_p)

corr_s_img2 = correlation(img2, kernel_p)
conv_s_img2 = convolution_manual(img2, kernel_p)

fig, axis = plt.subplots(2, 2, figsize=(10, 10))

axis[0, 0].imshow(corr_s_img1, cmap='gray')
axis[0, 0].set_title('Correlation Image 1')
axis[0, 0].axis("off")

axis[0, 1].imshow(conv_s_img1, cmap='gray')
axis[0, 1].set_title('Convolucion Image 1')
axis[0, 1].axis("off")

axis[1, 0].imshow(corr_s_img2, cmap='gray')
axis[1, 0].set_title('Correlation Image 2')
axis[1, 0].axis("off")

axis[1, 1].imshow(conv_s_img2, cmap='gray')
axis[1, 1].set_title('Convolucion Image 2')
axis[1, 1].axis("off")

plt.tight_layout()
plt.show()


# ==============================================================================
# 3. EXPERIMENTO B: KERNEL ASIMÉTRICO (DERIVADA HORIZONTAL)
# ==============================================================================
# Definir kernel asimétrico de derivada horizontal: [1, 0, -1]
kernel_as = np.array([[1, 0, -1]], dtype=np.float64)

# Embeber en una matriz 3x3
kernel_ac = np.zeros((3, 3), dtype=np.float64)
kernel_ac[1, :] = kernel_as  # Reemplazar fila central

# Convertir a float64 antes de procesar para admitir valores negativos
img1_f = img1.astype(np.float64)
img2_f = img2.astype(np.float64)

corr_as_img1 = correlation(img1_f, kernel_ac)
conv_as_img1 = convolution_manual(img1_f, kernel_ac)

corr_as_img2 = correlation(img2_f, kernel_ac)
conv_as_img2 = convolution_manual(img2_f, kernel_ac)

fig, axis = plt.subplots(2, 2, figsize=(10, 10))

axis[0, 0].imshow(corr_as_img1, cmap='gray')
axis[0, 0].set_title('Correlacion-Img1 (Asimétrica)')
axis[0, 0].axis('off')

axis[0, 1].imshow(conv_as_img1, cmap='gray')
axis[0, 1].set_title('Convolucion-Img1 (Invertida 180°)')
axis[0, 1].axis('off')

axis[1, 0].imshow(corr_as_img2, cmap='gray')
axis[1, 0].set_title('Correlacion-Img2 (Asimétrica)')
axis[1, 0].axis('off')

axis[1, 1].imshow(conv_as_img2, cmap='gray')
axis[1, 1].set_title('Convolucion-Img2 (Invertida 180°)')
axis[1, 1].axis('off')

plt.tight_layout()
plt.show()


# ==============================================================================
# 4. EXPERIMENTO C: MODOS DE BORDER PADDING (cv2.copyMakeBorder)
# ==============================================================================
borde = 20

# Padding sobre img1 (Cameraman)
img1_zero = cv2.copyMakeBorder(img1, borde, borde, borde, borde, cv2.BORDER_CONSTANT, value=0)
img1_replica = cv2.copyMakeBorder(img1, borde, borde, borde, borde, cv2.BORDER_REPLICATE)
img1_reflect = cv2.copyMakeBorder(img1, borde, borde, borde, borde, cv2.BORDER_REFLECT)

# Padding sobre img2 (Coins)
img2_zero = cv2.copyMakeBorder(img2, borde, borde, borde, borde, cv2.BORDER_CONSTANT, value=0)
img2_replica = cv2.copyMakeBorder(img2, borde, borde, borde, borde, cv2.BORDER_REPLICATE)
img2_reflect = cv2.copyMakeBorder(img2, borde, borde, borde, borde, cv2.BORDER_REFLECT)

fig, axis = plt.subplots(2, 3, figsize=(12, 8))

axis[0, 0].imshow(img1_zero, cmap='gray')
axis[0, 0].set_title('Cero img 1 (Constant 0)')
axis[0, 0].axis('off')

axis[0, 1].imshow(img1_replica, cmap='gray')
axis[0, 1].set_title('Replicar img 1 (Replicate)')
axis[0, 1].axis('off')

axis[0, 2].imshow(img1_reflect, cmap='gray')
axis[0, 2].set_title('Reflejar img 1 (Reflect)')
axis[0, 2].axis('off')

axis[1, 0].imshow(img2_zero, cmap='gray')
axis[1, 0].set_title('Cero Img 2 (Constant 0)')
axis[1, 0].axis('off')

axis[1, 1].imshow(img2_replica, cmap='gray')
axis[1, 1].set_title('Replicar Img 2 (Replicate)')
axis[1, 1].axis('off')

axis[1, 2].imshow(img2_reflect, cmap='gray')
axis[1, 2].set_title('Reflejar Img 2 (Reflect)')
axis[1, 2].axis('off')

plt.tight_layout()
plt.show()
```

---

## 🔍 Explicación de los resultados visuales

### 1. Kernel Simétrico: Correlación $\equiv$ Convolución
![[filtrado-kernel-simetrico-comparacion.png]]

- Debido a que $w(s, t) = w(-s, -t)$, rotar la máscara $180^\circ$ no altera los coeficientes. La correlación y la convolución producen matrices idénticas.

### 2. Kernel Asimétrico: Inversión de signos
![[filtrado-kernel-asimetrico-comparacion.png]]

- Al usar el kernel diferencial $[1, 0, -1]$, su rotado de 180° es $[-1, 0, 1]$.
- Esto invierte la dirección de la derivada espacial: las zonas claras en correlación se vuelven oscuras en convolución, y viceversa.

### 3. Comparativa de Padding
![[padding-modos-comparacion-visual.png]]

- **Constant (0):** Introduce un borde negro artificial.
- **Replicate:** Extiende los píxeles perimetrales (útil en imágenes con fondo uniforme).
- **Reflect:** Proporciona simetría y continuidad suave de gradientes en todas las direcciones.

---

## 🔗 Relacionado

- [[Filtrado Espacial y Convolución]] — formulación teórica
- [[Manejo de Bordes y Padding en Imágenes]] — fundamentos de frontera
- [[2026-09-15 - Implementación de Convolución y Modos de Borde]] — clase de laboratorio
- [[Inicio]] — mapa general de la materia

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #scipy #matplotlib #filtrado-espacial #convolución #padding

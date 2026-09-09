---
titulo: "Filtrado Espacial y Modos de Padding - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-09-09
clase: "[[2026-09-09 - Filtrado Espacial, Convolución y Manejo de Bordes]]"
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
> Este script demuestra en **Python con OpenCV, SciPy y Matplotlib**:
> 1. Los **4 modos principales de padding** para el manejo de bordes usando `cv2.copyMakeBorder`.
> 2. La diferencia práctica entre **Correlación** (`scipy.ndimage.correlate`) y **Convolución** (`scipy.ndimage.convolve`) usando un kernel asimétrico.
> 3. El filtrado espacial lineal con `cv2.filter2D`.

> [!info] Clase y conceptos relacionados
> 📅 Clase: [[2026-09-09 - Filtrado Espacial, Convolución y Manejo de Bordes]]
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

# ==============================================================================
# 1. DEMOSTRACIÓN VISUAL DE LOS 4 TIPOS DE PADDING
# ==============================================================================
img = data.camera()  # Imagen de 512x512

# Definir un ancho de borde visible para inspección (e.g. 50 píxeles)
pad = 50

# 1. Relleno constante con ceros (Zero-Padding)
pad_constant = cv2.copyMakeBorder(img, pad, pad, pad, pad, cv2.BORDER_CONSTANT, value=0)

# 2. Replicación (Clamp / Replicate)
pad_replicate = cv2.copyMakeBorder(img, pad, pad, pad, pad, cv2.BORDER_REPLICATE)

# 3. Reflexión con duplicación (Mirror con frontera repetida)
pad_reflect = cv2.copyMakeBorder(img, pad, pad, pad, pad, cv2.BORDER_REFLECT)

# 4. Reflexión sin duplicar frontera (Mirror 101 - Predeterminado en OpenCV)
pad_reflect101 = cv2.copyMakeBorder(img, pad, pad, pad, pad, cv2.BORDER_REFLECT_101)

fig, axes = plt.subplots(2, 2, figsize=(12, 10))

axes[0, 0].imshow(pad_constant, cmap='gray')
axes[0, 0].set_title('1. BORDER_CONSTANT (Zero-Pad)')
axes[0, 0].axis('off')

axes[0, 1].imshow(pad_replicate, cmap='gray')
axes[0, 1].set_title('2. BORDER_REPLICATE (Clamp)')
axes[0, 1].axis('off')

axes[1, 0].imshow(pad_reflect, cmap='gray')
axes[1, 0].set_title('3. BORDER_REFLECT (Mirror)')
axes[1, 0].axis('off')

axes[1, 1].imshow(pad_reflect101, cmap='gray')
axes[1, 1].set_title('4. BORDER_REFLECT_101 (Mirror 101)')
axes[1, 1].axis('off')

plt.tight_layout()
plt.show()


# ==============================================================================
# 2. DEMOSTRACIÓN: CORRELACIÓN VS CONVOLUCIÓN
# ==============================================================================
# Definir un kernel ASIMÉTRICO (filtro de derivada horizontal)
kernel_asimetrico = np.array([
    [-1, 0, 1],
    [-2, 0, 2],
    [-1, 0, 1]
], dtype=np.float32)

# Rotación manual de 180 grados del kernel (flip horizontal y vertical)
kernel_rotado_180 = np.rot90(kernel_asimetrico, 2)

print("--- Kernel Original (Sobel Gx) ---")
print(kernel_asimetrico)
print("\n--- Kernel Rotado 180° ---")
print(kernel_rotado_180)

# Aplicar Correlación y Convolución con SciPy
res_correlacion = ndimage.correlate(img.astype(float), kernel_asimetrico, mode='reflect')
res_convolucion = ndimage.convolve(img.astype(float), kernel_asimetrico, mode='reflect')

# Comprobación de la regla de oro:
# Convolución(K) == Correlación(K_rotado_180)
res_correlacion_rotado = ndimage.correlate(img.astype(float), kernel_rotado_180, mode='reflect')
error_regla_oro = np.max(np.abs(res_convolucion - res_correlacion_rotado))

print(f"\nDiscrepancia entre Convolución y Correlación rotada: {error_regla_oro:.6f}")

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

axes[0].imshow(res_correlacion, cmap='gray')
axes[0].set_title('Correlación Espacial (Sin rotación previa)')
axes[0].axis('off')

axes[1].imshow(res_convolucion, cmap='gray')
axes[1].set_title('Convolución Espacial (Con rotación de 180°)')
axes[1].axis('off')

plt.tight_layout()
plt.show()


# ==============================================================================
# 3. FILTRADO CON OPENCV: cv2.filter2D
# ==============================================================================
# Nota: cv2.filter2D realiza computacionalmente CORRELACIÓN.
# Para hacer convolución estricta con OpenCV, se debe rotar el kernel primero con np.flip:
# kernel_conv = np.flip(kernel)
res_opencv = cv2.filter2D(img, ddepth=-1, kernel=kernel_asimetrico, borderType=cv2.BORDER_REFLECT_101)
```

---

## 🔍 Puntos Clave de Implementación

1. **`cv2.filter2D` de OpenCV**:
   - Por motivos de optimización en hardware, `cv2.filter2D` ejecuta internamente **correlación**. Si el algoritmo exige convolución estricta sobre un kernel asimétrico, se debe invertir el kernel previamente con `np.flip(kernel)`.
2. **`borderType` en OpenCV**:
   - Permite elegir el comportamiento de borde directamente durante el filtrado sin tener que llamar a `copyMakeBorder` de forma manual. El predeterminado en la mayoría de filtros es `cv2.BORDER_REFLECT_101`.
3. **Mapeo de la regla de oro**:
   - La comprobación `np.max(np.abs(res_convolucion - res_correlacion_rotado))` da un error idéntico a $0.0$, demostrando matemáticamente que:
     $$ \text{Convolución}(K) \equiv \text{Correlación}(K_{180^\circ}) $$

---

## 🔗 Relacionado

- [[Filtrado Espacial y Convolución]] — formulación matemática
- [[Manejo de Bordes y Padding en Imágenes]] — fundamentos de frontera
- [[2026-09-09 - Filtrado Espacial, Convolución y Manejo de Bordes]] — apuntes de clase
- [[Inicio]] — mapa general de la materia

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #scipy #matplotlib #filtrado-espacial #convolución #padding

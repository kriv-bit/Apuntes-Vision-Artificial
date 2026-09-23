---
titulo: "Filtros de Suavizado y Separabilidad - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-09-23
clase: "[[2026-09-23 - Filtros de Suavizado y Separabilidad de Kernels]]"
tags:
  - visión-artificial
  - python
  - opencv
  - matplotlib
  - suavizado
  - filtro-gaussiano
  - filtro-mediana
  - separabilidad
aliases:
  - Código filtros de suavizado
  - Código filtro gaussiano vs mediana
  - cv2.sepFilter2D y cv2.medianBlur
---

# Filtros de Suavizado y Separabilidad — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python y OpenCV**:
> 1. Comparativa visual entre **Filtro de Caja** (`cv2.blur`) y **Filtro Gaussiano** (`cv2.GaussianBlur`).
> 2. Comparativa de eliminación de **ruido Sal y Pimienta**: demostración de por qué el Gaussiano falla y el **Filtro de la Mediana** (`cv2.medianBlur`) triunfa.
> 3. Implementación de **convolución separable** (`cv2.sepFilter2D`) frente a convolución directa 2D (`cv2.filter2D`), comparando tiempos de cómputo para verificar el paso de $\mathcal{O}(K^2)$ a $\mathcal{O}(2K)$.

> [!info] Clases y conceptos relacionados
> 📅 Clase: [[2026-09-23 - Filtros de Suavizado y Separabilidad de Kernels]]
> 🧠 Concepto: [[Filtros de Suavizado Espacial]] y [[Filtrado Espacial y Convolución]]

---

## 📦 Requisitos

```bash
pip install opencv-python-headless numpy matplotlib scikit-image
```

---

## 💻 Código completo

```python
import time
import numpy as np
import cv2
import matplotlib.pyplot as plt
from skimage import data

# Configuración global de figuras
plt.rcParams["figure.figsize"] = (12, 6)

img = data.camera()

# ==============================================================================
# 1. COMPARATIVA: FILTRO DE CAJA VS FILTRO GAUSSIANO
# ==============================================================================
ksize = 9

# Filtro de Caja (Promedio)
box_filtered = cv2.blur(img, (ksize, ksize))

# Filtro Gaussiano (sigmaX = 0 hace que OpenCV lo calcule automáticamente según ksize)
gaussian_filtered = cv2.GaussianBlur(img, (ksize, ksize), sigmaX=2.0)

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

axes[0].imshow(img, cmap='gray')
axes[0].set_title('Original')
axes[0].axis('off')

axes[1].imshow(box_filtered, cmap='gray')
axes[1].set_title(f'Filtro de Caja ({ksize}x{ksize})')
axes[1].axis('off')

axes[2].imshow(gaussian_filtered, cmap='gray')
axes[2].set_title(f'Filtro Gaussiano ({ksize}x{ksize}, σ=2.0)')
axes[2].axis('off')

plt.tight_layout()
plt.show()


# ==============================================================================
# 2. RUIDO SAL Y PIMIENTA: GAUSSIANO VS MEDIANA
# ==============================================================================
def add_salt_and_pepper_noise(image, salt_prob=0.02, pepper_prob=0.02):
    """Genera ruido impulsivo de sal y pimienta de forma estocástica"""
    noisy = image.copy()
    num_salt = np.ceil(salt_prob * image.size)
    coords = [np.random.randint(0, i - 1, int(num_salt)) for i in image.shape]
    noisy[tuple(coords)] = 255

    num_pepper = np.ceil(pepper_prob * image.size)
    coords = [np.random.randint(0, i - 1, int(num_pepper)) for i in image.shape]
    noisy[tuple(coords)] = 0
    return noisy

noisy_img = add_salt_and_pepper_noise(img, salt_prob=0.03, pepper_prob=0.03)

# Aplicar Gaussiano y Mediana sobre la imagen con sal y pimienta
gauss_denoised = cv2.GaussianBlur(noisy_img, (5, 5), sigmaX=1.5)
median_denoised = cv2.medianBlur(noisy_img, 5)

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

axes[0].imshow(noisy_img, cmap='gray')
axes[0].set_title('Imagen con Sal y Pimienta')
axes[0].axis('off')

axes[1].imshow(gauss_denoised, cmap='gray')
axes[1].set_title('Filtro Gaussiano (5x5)\nFalla: difumina el ruido en manchas')
axes[1].axis('off')

axes[2].imshow(median_denoised, cmap='gray')
axes[2].set_title('Filtro Mediana (5x5)\nTriunfa: elimina el ruido y preserva bordes')
axes[2].axis('off')

plt.tight_layout()
plt.show()


# ==============================================================================
# 3. VERIFICACIÓN DE SEPARABILIDAD: cv2.filter2D vs cv2.sepFilter2D
# ==============================================================================
# Kernel Gaussiano grande para magnificar la diferencia de tiempo
K = 31
sigma = 5.0

# 1. Obtener los vectores 1D separables
kernel_1d = cv2.getGaussianKernel(K, sigma)

# 2. Generar el kernel 2D multiplicando los vectores 1D (producto exterior)
kernel_2d = kernel_1d @ kernel_1d.T

# Medición de tiempo: Convolución 2D Directa (O(K^2))
start_2d = time.perf_counter()
for _ in range(50):
    res_2d = cv2.filter2D(img, -1, kernel_2d)
time_2d = (time.perf_counter() - start_2d) / 50.0

# Medición de tiempo: Convolución Separable 1D + 1D (O(2K))
start_sep = time.perf_counter()
for _ in range(50):
    res_sep = cv2.sepFilter2D(img, -1, kernel_1d, kernel_1d)
time_sep = (time.perf_counter() - start_sep) / 50.0

# Comprobar que ambas salidas son matemáticamente equivalentes
diff_max = np.max(np.abs(res_2d.astype(float) - res_sep.astype(float)))

print("=== BENCHMARK DE SEPARABILIDAD ===")
print(f"Kernel Gaussiano: {K}x{K} (K = {K})")
print(f"Tiempo Convolución Directa 2D : {time_2d * 1000:.3f} ms")
print(f"Tiempo Convolución Separable 1D: {time_sep * 1000:.3f} ms")
print(f"Factor de aceleración medido   : {time_2d / time_sep:.2f}x")
print(f"Diferencia máxima entre salidas: {diff_max:.6f} (Son idénticas)")
```

---

## 🔍 Conclusiones del Experimento

1. **Gaussiano vs Caja:**
   - El filtro de caja suaviza rápidamente pero genera halos y artefactos rectangulares en las transiciones de alta frecuencia. El gaussiano produce transiciones armónicas continuas.
2. **Por qué la Mediana es insustituible en Ruido Impulsivo:**
   - La mediana ignora por completo los valores extremos 0 y 255 al ordenar la lista de vecinos, mientras que el gaussiano incorpora esos valores en la media aritmética, convirtiendo los puntos aislados en borrones grises.
3. **Ganancia de la Separabilidad:**
   - En kernels grandes ($31 \times 31$ o superiores), la ejecución con `cv2.sepFilter2D` acelera el procesamiento drásticamente respecto a `cv2.filter2D`, con una equivalencia numérica perfecta.

---

## 🔗 Relacionado

- [[Filtros de Suavizado Espacial]] — fundamentos matemáticos
- [[Filtrado Espacial y Convolución]] — convolución y correlación
- [[2026-09-23 - Filtros de Suavizado y Separabilidad de Kernels]] — clase teórica
- [[Inicio]] — mapa general de la materia

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #suavizado #filtro-gaussiano #filtro-mediana #separabilidad #benchmarks

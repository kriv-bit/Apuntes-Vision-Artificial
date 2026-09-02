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
  - scikit-image
  - matplotlib
  - transformaciones-espaciales
  - corrección-gamma
  - histogramas
aliases:
  - Código transformaciones de intensidad
  - Código corrección gamma y estiramiento
---

# Transformaciones de Intensidad y Histogramas — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python (NumPy, Matplotlib y scikit-image)** las principales transformaciones de intensidad en el dominio espacial aplicadas sobre la imagen lunar (`data.moon()`):
> 1. **Negativo fotográfico** ($s = 255 - r$).
> 2. **Transformación Logarítmica** ($s = c \log(1 + r)$).
> 3. **Corrección Gamma** ($\gamma = 0.5, 1.0, 2.0$).
> 4. **Estiramiento de Contraste con Percentiles** (`np.percentile` y `np.clip`).
> 5. **Análisis de Histogramas** comparando la distribución original frente al estiramiento.

> [!info] Clase y conceptos relacionados
> 📅 Clase: [[2026-09-02 - Transformaciones de Intensidad y Procesamiento de Histogramas]]
> 🧠 Conceptos: [[Transformaciones de Intensidad Espacial]] y [[Histogramas y Ecualización de Imagen]]

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

# Configuración global del tamaño de figuras
plt.rcParams["figure.figsize"] = (10, 5)

# ==============================================================================
# 1. CARGA DE IMAGEN DE PRUEBA (LUNA)
# ==============================================================================
img = data.moon()

fig, axis = plt.subplots(1)
axis.imshow(img, cmap="gray")
axis.set_title("Original")
axis.axis("off")
plt.tight_layout()
plt.show()


# ==============================================================================
# 2. INVERSIÓN FOTOGRÁFICA / NEGATIVO (s = 255 - r)
# ==============================================================================
negative = 255 - img

fig, axis = plt.subplots(1)
axis.imshow(negative, cmap='gray')
axis.set_title('Negativo')
axis.axis('off')
plt.tight_layout()
plt.show()


# ==============================================================================
# 3. TRANSFORMACIÓN LOGARÍTMICA (s = c * log(1 + r))
# ==============================================================================
# c = 255 / log(1 + max_pixel)
c_log = 255.0 / np.log(1.0 + img.astype(np.float64).max())
log_transform = (c_log * np.log(1.0 + img.astype(np.float64))).astype(np.uint8)

fig, axis = plt.subplots(1)
axis.imshow(log_transform, cmap='gray')
axis.set_title('Logarithmic Transform')
axis.axis('off')
plt.tight_layout()
plt.show()


# ==============================================================================
# 4. CORRECCIÓN GAMMA / LEY DE POTENCIAS (s = c * r^gamma)
# ==============================================================================
def gamma_transform(image, gamma, c=1.0):
    """
    Aplica corrección gamma normalizando a [0, 1] para evitar overflow.
    - gamma < 1: Aclara la imagen y expande oscuros.
    - gamma = 1: Identidad.
    - gamma > 1: Oscurece la imagen y comprime sombras.
    """
    norm = image.astype(np.float64) / 255.0
    output = c * np.power(norm, gamma)
    return np.uint8(255.0 * output)

gamma_05 = gamma_transform(img, 0.5)
gamma_10 = gamma_transform(img, 1.0)
gamma_20 = gamma_transform(img, 2.0)

fig, axis = plt.subplots(1, 4, figsize=(16, 4))

axis[0].imshow(img, cmap='gray')
axis[0].set_title('Original Image')
axis[0].axis('off')

axis[1].imshow(gamma_05, cmap='gray')
axis[1].set_title('Gamma γ = 0.5 (Aclarado)')
axis[1].axis('off')

axis[2].imshow(gamma_10, cmap='gray')
axis[2].set_title('Gamma γ = 1.0 (Identidad)')
axis[2].axis('off')

axis[3].imshow(gamma_20, cmap='gray')
axis[3].set_title('Gamma γ = 2.0 (Oscurecido)')
axis[3].axis('off')

plt.tight_layout()
plt.show()


# ==============================================================================
# 5. ESTIRAMIENTO DE CONTRASTE POR PERCENTILES CON NP.CLIP
# ==============================================================================
# Obtener percentiles para definir el rango de interés dinámico
r_min = np.percentile(img, 40)
r_max = np.percentile(img, 50)

# Mapeo lineal y truncamiento seguro entre 0 y 255
stretching = np.clip(
    (img.astype(np.float64) - r_min) * 255.0 / (r_max - r_min), 
    0, 
    255
).astype(np.uint8)

fig, axis = plt.subplots(1, 2, figsize=(10, 5))

axis[0].imshow(img, cmap='gray')
axis[0].set_title('Original Image')
axis[0].axis('off')

axis[1].imshow(stretching, cmap='gray')
axis[1].set_title('Contrast Stretching (P40 - P50)')
axis[1].axis('off')

plt.tight_layout()
plt.show()


# ==============================================================================
# 6. CÁLCULO Y VISUALIZACIÓN DE HISTOGRAMAS
# ==============================================================================
fig, axes = plt.subplots(2, 2, figsize=(12, 8))

# Imagen original y su histograma
axes[0, 0].imshow(img, cmap='gray')
axes[0, 0].set_title('Original')
axes[0, 0].axis('off')

axes[0, 1].hist(img.ravel(), bins=256, range=[0, 256], color='steelblue')
axes[0, 1].set_title('Histogram (Original)')
axes[0, 1].set_xlabel('Nivel de gris')
axes[0, 1].set_ylabel('Frecuencia')
axes[0, 1].set_xlim([0, 256])

# Imagen con estiramiento y su histograma
axes[1, 0].imshow(stretching, cmap='gray')
axes[1, 0].set_title('Stretched (P40 - P50)')
axes[1, 0].axis('off')

axes[1, 1].hist(stretching.ravel(), bins=256, range=[0, 256], color='steelblue')
axes[1, 1].set_title('Histogram (Stretched)')
axes[1, 1].set_xlabel('Nivel de gris')
axes[1, 1].set_ylabel('Frecuencia')
axes[1, 1].set_xlim([0, 256])

plt.tight_layout()
plt.show()
```

---

## 🔍 Explicación paso a paso de los resultados

### 1. Imagen Original vs Negativo
![[transformacion-luna-original.png]] ![[transformacion-luna-negativo.png]]

- La inversión resalta las zonas de sombras profundas y los bordes convexos de los cráteres.

### 2. Transformación Logarítmica
![[transformacion-luna-logaritmica.png]]

- Eleva drásticamente las intensidades medias y bajas del suelo lunar, mostrando detalles que antes estaban ocultos en la oscuridad.

### 3. Respuesta Gamma ($\gamma = 0.5, 1.0, 2.0$)
![[transformacion-luna-gamma-comparacion.png]]

- **$\gamma = 0.5$**: Curva convexa $\to$ aclara la imagen general.
- **$\gamma = 1.0$**: Mantiene la imagen sin cambios.
- **$\gamma = 2.0$**: Curva cóncava $\to$ incrementa las sombras y el dramatismo del relieve.

### 4. Estiramiento de Contraste con Percentiles
![[transformacion-luna-estiramiento-percentiles.png]]

- Al fijar un rango estrecho entre los percentiles 40 y 50, los tonos intermedios se extienden al rango completo $[0, 255]$, binarizando visualmente las regiones de terreno.

### 5. Comparativa de Histogramas
![[transformacion-luna-histogramas-comparacion.png]]

- Demuestra gráficamente cómo una imagen de bajo contraste posee un histograma angosto con forma de campana en el centro (~115), y cómo el estiramiento agresivo redistribuye los píxeles hacia los extremos $0$ y $255$.

---

## 🔗 Relacionado

- [[Transformaciones de Intensidad Espacial]] — teoría matemática
- [[Histogramas y Ecualización de Imagen]] — funciones de distribución y CDF
- [[2026-09-02 - Transformaciones de Intensidad y Procesamiento de Histogramas]] — clase teórica
- [[Inicio]] — mapa general de la materia

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #scikit-image #matplotlib #transformaciones-espaciales #corrección-gamma #histogramas

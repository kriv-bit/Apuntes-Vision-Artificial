---
titulo: "Ecualización y Especificación de Histogramas - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-09-08
clase: "[[2026-09-08 - Ecualización y Especificación de Histogramas]]"
tags:
  - visión-artificial
  - python
  - opencv
  - scikit-image
  - matplotlib
  - histogramas
  - ecualización
  - histogram-matching
aliases:
  - Código ecualización de histograma
  - Código histogram matching Python
  - skimage exposure match_histograms
---

# Ecualización y Especificación de Histogramas — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python con OpenCV, scikit-image y Matplotlib**:
> 1. Creación sintética de una imagen de muy bajo contraste mediante corrección gamma ($\gamma = 3.0$).
> 2. **Ecualización global de histograma** con `cv2.equalizeHist()`.
> 3. **Especificación / Emparejamiento de histograma (*Histogram Matching*)** con `skimage.exposure.match_histograms()` para adaptar el contraste y tonalidad de una imagen a partir de otra de referencia.

> [!info] Clase y conceptos relacionados
> 📅 Clase: [[2026-09-08 - Ecualización y Especificación de Histogramas]]
> 🧠 Concepto: [[Histogramas y Ecualización de Imagen]] y [[Transformaciones de Intensidad Espacial]]

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
from skimage import data, exposure

# Configuración global de figuras
plt.rcParams["figure.figsize"] = (10, 6)

# ==============================================================================
# 1. GENERACIÓN DE IMAGEN CON BAJO CONTRASTE
# ==============================================================================
img = data.moon()

def gamma_transform(image, gamma, c=1.0):
    """Aplica corrección gamma normalizando a [0, 1]"""
    norm = image.astype(np.float64) / 255.0
    output = c * np.power(norm, gamma)
    return np.uint8(255.0 * output)

# Una gamma alta (3.0) comprime las intensidades al extremo oscuro
img_low_contrast = gamma_transform(img, 3.0)


# ==============================================================================
# 2. ECUALIZACIÓN GLOBAL DE HISTOGRAMA CON OPENCV
# ==============================================================================
equalized = cv2.equalizeHist(img_low_contrast)

# Visualización comparativa 2x2: Imagen e Histograma
fig, axis = plt.subplots(2, 2, figsize=(12, 8))

# Fila 1: Imagen de bajo contraste y su histograma
axis[0][0].imshow(img_low_contrast, cmap='gray')
axis[0][0].set_title('Low Contrast (Gamma = 3.0)')
axis[0][0].axis('off')

axis[0][1].hist(img_low_contrast.ravel(), bins=256, range=(0, 256), color="steelblue")
axis[0][1].set_title('Histogram (Low Contrast)')
axis[0][1].set_xlabel('Nivel de gris')
axis[0][1].set_ylabel('Frecuencia')
axis[0][1].set_xlim([0, 256])

# Fila 2: Imagen ecualizada y su histograma
axis[1][0].imshow(equalized, cmap='gray')
axis[1][0].set_title('Equalized (cv2.equalizeHist)')
axis[1][0].axis('off')

axis[1][1].hist(equalized.ravel(), bins=256, range=(0, 256), color="steelblue")
axis[1][1].set_title('Histogram (Equalized)')
axis[1][1].set_xlabel('Nivel de gris')
axis[1][1].set_ylabel('Frecuencia')
axis[1][1].set_xlim([0, 256])

plt.tight_layout()
plt.show()


# ==============================================================================
# 3. ESPECIFICACIÓN / MATCHING DE HISTOGRAMA CON SCIKIT-IMAGE
# ==============================================================================
# Cargamos la imagen de referencia (Cameraman)
ref_img = data.camera()

# Ajustamos el histograma de la luna para que se parezca al del cameraman
match_img = exposure.match_histograms(img_low_contrast, ref_img)

# Visualización comparativa 3x2: Origen, Referencia y Resultado
fig, axis = plt.subplots(3, 2, figsize=(12, 11))

# Fila 1: Imagen origen de bajo contraste
axis[0][0].imshow(img_low_contrast, cmap='gray')
axis[0][0].set_title('Low Contrast')
axis[0][0].axis('off')

axis[0][1].hist(img_low_contrast.ravel(), bins=256, range=(0, 256), color="steelblue")
axis[0][1].set_title('Histogram (Low Contrast)')
axis[0][1].set_xlabel('Nivel de gris')
axis[0][1].set_ylabel('Frecuencia')
axis[0][1].set_xlim([0, 256])

# Fila 2: Imagen de referencia (Cameraman)
axis[1][0].imshow(ref_img, cmap='gray')
axis[1][0].set_title('Ref Imagen (Cameraman)')
axis[1][0].axis('off')

axis[1][1].hist(ref_img.ravel(), bins=256, range=(0, 256), color="steelblue")
axis[1][1].set_title('Histogram (Referencia)')
axis[1][1].set_xlabel('Nivel de gris')
axis[1][1].set_ylabel('Frecuencia')
axis[1][1].set_xlim([0, 256])

# Fila 3: Imagen resultante con histograma acoplado
axis[2][0].imshow(match_img, cmap='gray')
axis[2][0].set_title('Imagen resultante (Matched)')
axis[2][0].axis('off')

axis[2][1].hist(match_img.ravel(), bins=256, range=(0, 256), color="steelblue")
axis[2][1].set_title('Histogram (Resultado Matched)')
axis[2][1].set_xlabel('Nivel de gris')
axis[2][1].set_ylabel('Frecuencia')
axis[2][1].set_xlim([0, 256])

plt.tight_layout()
plt.show()
```

---

## 🔍 Explicación paso a paso de los resultados

### 1. Ecualización frente a Bajo Contraste
![[histograma-ecualizacion-luna-comparacion.png]]

- **Histograma original:** Concentrado entre $0$ y $35$. Toda la información está aprisionada en las sombras profundas.
- **Histograma ecualizado:** Las barras se distribuyen espaciadas a lo largo de todo el espectro de 8 bits ($0$ a $255$). Al aplicar la CDF, los píxeles intermedios saltan a niveles más luminosos, revelando el relieve de los cráteres.

---

### 2. Matching de Histogramas con Imagen de Referencia
![[histograma-matching-especificacion-comparacion.png]]

- La función `exposure.match_histograms(source, reference)` calcula la CDF de ambas imágenes y mapea las intensidades de `source` para reproducir la distribución de `reference`.
- Fíjate en los 3 picos del histograma de la imagen de referencia:
  1. Pico bajo (abrigo del fotógrafo).
  2. Meseta intermedia (césped).
  3. Pico alto (cielo).
- El histograma de la imagen resultante adopta exactamente ese perfil trimodal, produciendo un aspecto tonal coherente con la escena de referencia.

---

## ⚠️ Errores comunes

1. **Confundir dimensiones en el matching:**
   - A diferencia de las operaciones aritméticas (`img1 + img2`), `match_histograms` **no exige** que ambas imágenes tengan las mismas dimensiones ($M \times N$), porque opera sobre las funciones de probabilidad acumulada (normalizadas por el total de píxeles).
2. **Aplicar ecualización a imágenes que ya tienen buen contraste:**
   - En imágenes balanceadas, la ecualización forzada puede aumentar el ruido de fondo (*over-enhancement*) o quemar las altas luces.
3. **Tipo de dato en `cv2.equalizeHist`:**
   - `cv2.equalizeHist` **solo admite** matrices en formato `np.uint8` de un solo canal (escala de grises). Si pasas una matriz `float64`, arrojará un error de tipo.

---

## 🔗 Relacionado

- [[Histogramas y Ecualización de Imagen]] — teoría matemática y formulación de CDF
- [[Transformaciones de Intensidad Espacial]] — transformaciones puntuales y gamma
- [[2026-09-08 - Ecualización y Especificación de Histogramas]] — clase teórica
- [[Inicio]] — mapa de contenidos

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #scikit-image #matplotlib #histogramas #ecualización #histogram-matching

---
titulo: "Reducción de Niveles de Gris - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-08-12
clase: "[[2026-08-12 - Fundamentos de Imágenes]]"
tags:
  - visión-artificial
  - python
  - numpy
  - matplotlib
  - niveles-de-gris
aliases:
  - Código cuantización de grises
---

# Reducción de Niveles de Gris — Python (Cuantización)

> [!info] ¿Qué es exactamente este código?
> Esto es **cuantización** (reducción de los niveles de intensidad de la imagen). **No es muestreo**: el muestreo define la **resolución espacial** (cuántos píxeles se capturan); la cuantización define la **resolución de intensidad** (cuántos niveles de gris). Ver [[Resolución de Imagen]].

> [!info] Clase y teoría relacionadas
> 📅 Clase: [[2026-08-12 - Fundamentos de Imágenes]]
> 🧠 Concepto: [[Reducción de Niveles de Gris]]

## 🎯 Qué hace el programa

1. Carga una imagen desde la PC (Google Colab) y la convierte a **escala de grises**.
2. Reduce los niveles de gris a **64, 20, 8 y 2 niveles** aplicando $nuevo\_valor = \left\lfloor \frac{pixel}{q} \right\rfloor \times q$.
3. Asocia cada versión con sus **bits por píxel** usando $k = \log_2(L)$.
4. Compara las **5 versiones** lado a lado (original + 4 reducidas).

## 📦 Requisitos

- Python 3 con `numpy`, `matplotlib`, `Pillow`.
- **Google Colab** por el uso de `files.upload()`. En local, reemplaza esa parte por la ruta del archivo (ver [[#⚠️ Errores comunes]]).

## 💻 Código completo

```python
import numpy as np
import matplotlib.pyplot as plt
from PIL import Image
from google.colab import files

# Subir Imagen desde la PC
uploaded = files.upload()
filename = list(uploaded.keys())[0]
img_color = Image.open(filename)
img_grey = img_color.convert('L')
img = np.array(img_grey).astype(float)
print('Image size w,h: ', img.shape)
print('Min intensity value: ', img.min())
print('Max intesnity value: ', img.max())


# ==============================================================================
# REDUCCIÓN DE NIVELES DE GRIS (CUANTIZACIÓN)
# ==============================================================================
# Para reducir una imagen de 256 niveles de intensidad (0 a 255) a L niveles,
# aplicamos un proceso de dos pasos: dividir para agrupar y multiplicar para escalar.
#
# EJEMPLO PARA 8 NIVELES:
# 1. Tamaño del paso (step):
#    Dividimos el rango total (256) entre el número de niveles deseados (L):
#    256 / 8 = 32. Cada nivel abarcará un rango de 32 valores de intensidad.
#
# 2. Asignación del nivel (mapeo):
#    Dividimos la intensidad del píxel entre el paso (32) y redondeamos hacia abajo (floor).
#    Ejemplo: píxel con valor 43 -> 43 / 32 = 1.34 -> floor(1.34) = Nivel 1.
#
# 3. Homogeneización / reescalado:
#    Si dejamos solo los números de nivel (0 a 7), la imagen se vería casi negra.
#    Para devolver los píxeles a una intensidad visible en el rango 0-255,
#    multiplicamos el nivel obtenido nuevamente por el paso (32).
#
#    Ejemplo de unificación de intensidades similares:
#    - Píxel A (200) -> floor(200 / 32) = 6 -> 6 * 32 = 192
#    - Píxel B (220) -> floor(220 / 32) = 6 -> 6 * 32 = 192
#
#    Lo que antes eran dos intensidades distintas (200 y 220) ahora se igualan a 192.
#    De esta forma eliminamos la heterogeneidad y la matriz pasa de tener hasta 256
#    valores posibles a contener únicamente L valores discretos.
# ==============================================================================

# Se nos pidió un procesamiento que ocupara 6 bits: 2^6 = L, por lo que L = 64
# Entonces nuestro procesamiento será de 64 niveles.
# Para 64 niveles dividimos el rango total: 256 / 64 = 4
img_64_L = np.floor(img/4)*4

# Reducción de niveles de gris a 20
# Para obtener 20 niveles, dividimos el rango total 256 entre 20 = 12.8, se aproxima.
# Esto nos da el 'paso' o 'step' necesario para agrupar los píxeles.
img_20_L = np.floor(img/13)*13

# Reducción de niveles de gris a 8
img_8_L = np.floor(img/32)*32

# Reducción de niveles de gris a 2
img_2_L = np.floor(img/128)*128


# Comparación
# Corregido: plt.subplots en plural para poder crear 5 columnas
fig, axis = plt.subplots(1, 5, figsize=(22, 5))

# Sabemos que la cantidad de niveles es 2^k (k = bits que ocupa cada píxel).
# Para encontrar k: L = 2^k -> 20 = 2^k -> k = log2(20) = 4.3219
# (20 niveles no es potencia de 2, así que se redondea a 5 bits)

# 1. Original (256 niveles)
axis[0].imshow(img, cmap='gray', vmin=0, vmax=255)
axis[0].set_title('Original Image - 255 Niveles: 8bit')
axis[0].axis('off')

# 2. 64 niveles (ocupan 6 bits)
axis[1].imshow(img_64_L, cmap='gray', vmin=0, vmax=255)
axis[1].set_title('Second Image - 64 Niveles: 6bit')
axis[1].axis('off')

# 3. 20 niveles
axis[2].imshow(img_20_L, cmap='gray', vmin=0, vmax=255)
axis[2].set_title('Third Image - 20 Niveles: 5bit')
axis[2].axis('off')

# 4. 8 niveles
axis[3].imshow(img_8_L, cmap='gray', vmin=0, vmax=255)
axis[3].set_title('Fourth Image - 8 Niveles: 3bit')
axis[3].axis('off')

# 5. 2 niveles
axis[4].imshow(img_2_L, cmap='gray', vmin=0, vmax=255)
axis[4].set_title('Fifth Image - 2 Niveles: 1bit')
axis[4].axis('off')

plt.tight_layout()
plt.show()
```

## 🔍 Explicación paso a paso

### 1. Carga y conversión a grises

```python
uploaded = files.upload()                 # dict {nombre_archivo: contenido}
filename = list(uploaded.keys())[0]       # primer archivo subido
img_color = Image.open(filename)          # imagen a color
img_grey = img_color.convert('L')         # 'L' = luminancia -> escala de grises
img = np.array(img_grey).astype(float)    # matriz de flotantes (necesaria para dividir)
```

- `.astype(float)` es clave: la división necesita decimales; con enteros (`uint8`) NumPy truncaría antes del `floor`.

### 2. Cuantización: la misma fórmula en 4 versiones

```python
img_64_L = np.floor(img/4)*4      # 64 niveles
img_20_L = np.floor(img/13)*13    # 20 niveles
img_8_L  = np.floor(img/32)*32    # 8 niveles
img_2_L  = np.floor(img/128)*128  # 2 niveles
```

El paso $q$ sale de $q = \frac{256}{L}$:

| Variable | $L$ (niveles) | Paso $q = 256/L$ | Valores posibles |
| :------- | :-----------: | :--------------: | :--------------: |
| `img_64_L` | 64 | 4 | 0, 4, 8, …, 252 |
| `img_20_L` | 20 | 13 (≈ 12.8) | 0, 13, 26, …, 247 |
| `img_8_L` | 8 | 32 | 0, 32, 64, …, 224 |
| `img_2_L` | 2 | 128 | 0 y 128 |

> [!warning] El paso de 20 niveles no es exacto
> $256/20 = 12.8$ no es entero; el código usa **13**. Entonces el máximo valor alcanzable es $\lfloor 255/13 \rfloor \times 13 = 247$, así que el rango queda 0–247 en 20 escalones.

### 3. Bits por píxel: $k = \log_2(L)$

La cantidad de niveles es $L = 2^k$, con $k$ = bits por píxel. Despejando: $k = \log_2(L)$.

| $L$ | $k = \log_2(L)$ | Título en la figura |
| :-: | :-------------: | :-----------------: |
| 256 | 8 | `8bit` |
| 64 | 6 | `6bit` |
| 20 | $\approx 4.32 \rightarrow$ 5 | `5bit` |
| 8 | 3 | `3bit` |
| 2 | 1 | `1bit` |

> [!tip] Relación con el tamaño de la imagen
> Con $k$ ya se puede calcular cuántos bits ocupa toda la imagen: $b = N \times M \times k$. Ver [[Tamaño de una Imagen Digital]].

### 4. Comparación visual

```python
fig, axis = plt.subplots(1, 5, figsize=(22, 5))
```

- Crea una fila de **5 subgráficas** indexadas con `axis[0..4]`.
- `imshow(..., cmap='gray', vmin=0, vmax=255)` fija el rango de visualización 0–255.
- `plt.tight_layout()` acomoda los títulos y `plt.show()` renderiza.

## ✅ Resultado esperado

| Columna | Imagen | Niveles | Bits | Efecto visual |
| :-----: | ------ | :-----: | :--: | ------------- |
| 0 | Original | 256 | 8 | Degradado suave |
| 1 | `img_64_L` | 64 | 6 | Muy parecida a la original |
| 2 | `img_20_L` | 20 | 5 | Posterización leve |
| 3 | `img_8_L` | 8 | 3 | Posterización clara (8 tonos) |
| 4 | `img_2_L` | 2 | 1 | Solo 2 tonos: negro (0) y gris medio (128) |

## ⚠️ Errores comunes

- **`plt.subplot` vs `plt.subplots`**: sin la *s*, `subplot` devuelve un solo objeto y `axis[0]` falla. Esta versión ya viene corregida con `plt.subplots`.
- **`files.upload()` solo funciona en Google Colab.** En local usa:
  ```python
  img_color = Image.open("ruta/de/tu/imagen.jpg")
  ```
- **Olvidar `.astype(float)`**: con matriz `uint8`, `img / 32` hace división entera y pierdes decimales antes del `floor`.
- **Confundir muestreo con cuantización**: este código reduce **niveles de intensidad** (cuantización); el **muestreo** (resolución espacial) sería cambiar la cantidad de píxeles (ej: redimensionar).
- **Guardar el resultado como imagen de 8 bits**: `img_64_L` es `float`; para exportarla usa `img_64_L.astype(np.uint8)` y `Image.fromarray(...)`.

## 🔗 Relacionado

- [[Reducción de Niveles de Gris]] — teoría del concepto
- [[Tamaño de una Imagen Digital]] — $b = N \times M \times k$ y $k = \log_2(L)$
- [[Resolución de Imagen]] — cuantización = resolución de intensidad
- [[2026-08-12 - Fundamentos de Imágenes]] — clase donde se vio
- [[Inicio]] — mapa de contenidos

## 🏷️ Etiquetas

#visión-artificial #python #numpy #matplotlib #niveles-de-gris

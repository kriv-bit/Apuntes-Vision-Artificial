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
  - Código reducción de grises
---

# Reducción de Niveles de Gris — Python

> [!info] Clase y teoría relacionadas
> 📅 Clase: [[2026-08-12 - Fundamentos de Imágenes]]
> 🧠 Concepto: [[Reducción de Niveles de Gris]]

## 🎯 Qué hace el programa

1. Carga una imagen desde la PC (Google Colab).
2. La convierte a **escala de grises**.
3. Reduce los niveles de gris a **8 niveles** ($q = 32$) y a **2 niveles** ($q = 128$).
4. Muestra las 3 versiones lado a lado para comparar.

## 📦 Requisitos

- Python 3 con `numpy`, `matplotlib`, `Pillow`
- **Google Colab** por el uso de `files.upload()`. En local, reemplaza esa parte por la ruta del archivo (ver [[#⚠️ Errores comunes]]).

## 💻 Código completo

```python
import numpy as np
import matplotlib.pyplot as plt
from PIL import Image
from google.colab import files


# --- Subir imagen desde la PC ---
uploaded = files.upload()                 # abre el diálogo para elegir el archivo
filename = list(uploaded.keys())[0]       # toma el nombre del archivo subido
img_color = Image.open(filename)          # abre la imagen (a color)
img_grey = img_color.convert('L')         # la convierte a escala de grises
img = np.array(img_grey).astype(float)    # la pasa a matriz de flotantes

print('Image size w,h: ', img.shape)
print('Min intensity value: ', img.min())
print('Max intesnity value: ', img.max())


# --- Reducción de niveles de gris ---
# Fórmula: nuevo_valor = floor(pixel / q) * q
img_0_1 = np.floor(img / 32) * 32    # q = 32  -> 256/32 = 8 niveles
img_2_1 = np.floor(img / 128) * 128  # q = 128 -> 256/128 = 2 niveles


# --- Comparación lado a lado ---
fig, axis = plt.subplots(1, 3, figsize=(15, 5))

axis[0].imshow(img, cmap='gray', vmin=0, vmax=255)
axis[0].set_title('Original - 256 niveles')
axis[0].axis('off')

axis[1].imshow(img_0_1, cmap='gray', vmin=0, vmax=255)
axis[1].set_title('8 niveles (q=32)')
axis[1].axis('off')

axis[2].imshow(img_2_1, cmap='gray', vmin=0, vmax=255)
axis[2].set_title('2 niveles (q=128)')
axis[2].axis('off')

plt.show()
```

## 🔍 Explicación paso a paso

### 1. Importación de librerías

```python
import numpy as np
import matplotlib.pyplot as plt
from PIL import Image
from google.colab import files
```

| Librería | Para qué sirve |
| -------- | -------------- |
| `numpy` (`np`) | Operaciones numéricas con matrices |
| `matplotlib.pyplot` (`plt`) | Graficar y mostrar imágenes |
| `PIL.Image` (Pillow) | Abrir y procesar imágenes |
| `google.colab.files` | Diálogo de subida de archivos (solo Colab) |

### 2. Carga y conversión a grises

```python
uploaded = files.upload()                 # dict {nombre_archivo: contenido}
filename = list(uploaded.keys())[0]       # primer archivo subido
img_color = Image.open(filename)          # imagen a color
img_grey = img_color.convert('L')         # 'L' = luminancia -> escala de grises
img = np.array(img_grey).astype(float)    # matriz de flotantes
```

- `files.upload()` devuelve un **diccionario**; con `list(...)[0]` tomamos el nombre del primer archivo.
- `.convert('L')` descarta el color y deja un solo canal de intensidad (0 = negro, 255 = blanco).
- `.astype(float)` es clave: la división que viene después necesita decimales, y con enteros (`uint8`) Python/NumPy truncaría antes de tiempo.

### 3. Información de la imagen

```python
print('Image size w,h: ', img.shape)
print('Min intensity value: ', img.min())
print('Max intesnity value: ', img.max())
```

- `img.shape` → `(alto, ancho)` — aunque el print diga `w,h`, el orden real de la matriz es (filas, columnas).
- `min()` y `max()` confirman que la imagen está en el rango 0–255.

### 4. Reducción de niveles de gris

```python
img_0_1 = np.floor(img / 32) * 32    # 8 niveles
img_2_1 = np.floor(img / 128) * 128  # 2 niveles
```

Es **exactamente la fórmula del apunte** aplicada a toda la matriz de una vez:

$$ nuevo\_valor = \left\lfloor \frac{pixel}{q} \right\rfloor \times q $$

- `np.floor(x / q)` = piso: divide, descarta los decimales y queda el **nivel**.
- `* q` = multiplica de nuevo para devolverle al píxel la intensidad completa de ese nivel.

| Variable | $q$ | Niveles resultantes | Valores posibles |
| :------- | :-: | :-----------------: | :--------------: |
| `img_0_1` | 32 | 8 | 0, 32, 64, 96, 128, 160, 192, 224 |
| `img_2_1` | 128 | 2 | 0 y 128 |

### 5. Comparación visual

```python
fig, axis = plt.subplots(1, 3, figsize=(15, 5))
```

- Crea una fila de **3 subgráficas**; `axis[0]`, `axis[1]` y `axis[2]` son cada una.
- `imshow(..., cmap='gray', vmin=0, vmax=255)` pinta la matriz en grises fijando el rango 0–255.
- `plt.show()` renderiza la figura.

## ✅ Resultado esperado

| Columna | Imagen | Efecto visual |
| :-----: | ------ | ------------- |
| 0 | Original (256 niveles) | Degradado suave de grises |
| 1 | 8 niveles ($q=32$) | Posterizada: se ven 8 tonos, saltos visibles |
| 2 | 2 niveles ($q=128$) | Solo 2 tonos: negro (0) y gris medio (128) |

## ⚠️ Errores comunes

> [!warning] `plt.subplot` vs `plt.subplots`
> En el código original estaba `plt.subplot(1, 3, ...)` (sin *s*). `subplot` devuelve **un solo** objeto de ejes, así que `axis[0]` falla. Para una fila de 3 gráficas que se indexan con `axis[0..2]` debe ser **`plt.subplots(1, 3, ...)`**.

- **`files.upload()` solo funciona en Google Colab.** En local, usa:
  ```python
  img_color = Image.open("ruta/de/tu/imagen.jpg")
  ```
- **Olvidar `.astype(float)`**: con una matriz `uint8`, `img / 32` hace división entera y pierdes decimales antes del `floor`.
- **Guardar el resultado como imagen de 8 bits**: `img_0_1` es `float`; para exportarla usa `img_0_1.astype(np.uint8)` y `Image.fromarray(...)`.

## 🔗 Relacionado

- [[Reducción de Niveles de Gris]] — teoría del concepto
- [[2026-08-12 - Fundamentos de Imágenes]] — clase donde se vio
- [[Inicio]] — mapa de contenidos de la materia

## 🏷️ Etiquetas

#visión-artificial #python #numpy #matplotlib #niveles-de-gris

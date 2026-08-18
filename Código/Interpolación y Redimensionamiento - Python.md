---
titulo: "Interpolación y Redimensionamiento - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-08-18
clase: "[[2026-08-18 - Interpolación y Redimensionamiento]]"
tags:
  - visión-artificial
  - python
  - opencv
  - matplotlib
  - interpolación
aliases:
  - Código interpolación OpenCV
  - Redimensionamiento con cv2.resize
---

# Interpolación y Redimensionamiento — Python (OpenCV)

> [!info] ¿Qué es este código?
> Este script demuestra el **redimensionamiento espacial** (upscaling a $3\times$) de una imagen digital y compara cómo los distintos algoritmos de **interpolación** en OpenCV (`cv2.resize`) estiman los píxeles faltantes en la nueva cuadrícula.

> [!info] Clase y teoría relacionadas
> 📅 Clase: [[2026-08-18 - Interpolación y Redimensionamiento]]
> 🧠 Concepto: [[Interpolación]]
> 📐 Teoría base: [[Resolución de Imagen]] y [[Tamaño de una Imagen Digital]]

## 🎯 Qué hace el programa

1. Obtiene las dimensiones de la imagen original (`shape[0]` para alto y `shape[1]` para ancho) y define un nuevo tamaño escalado ($3\times$).
2. Aplica **5 métodos de interpolación** disponibles en OpenCV:
   - `cv2.INTER_NEAREST` (Vecino más cercano)
   - `cv2.INTER_LINEAR` (Bilineal)
   - `cv2.INTER_CUBIC` (Bicúbica)
   - `cv2.INTER_NEAREST_EXACT` (Vecino más cercano exacto)
   - `cv2.INTER_LINEAR_EXACT` (Bilineal exacto)
3. Realiza un **recorte / zoom de inspección** (`[0:zoom, 0:zoom]`) en la esquina superior izquierda para apreciar a nivel de píxel las diferencias de suavizado y nitidez.
4. Grafica y compara los resultados en subplots de Matplotlib.

## 📦 Requisitos

- Python 3
- Librerías: `opencv-python` (`cv2`), `numpy`, `matplotlib`, `Pillow`

```bash
pip install opencv-python numpy matplotlib pillow
```

## 💻 Código completo

```python
import cv2
import matplotlib.pyplot as plt
import numpy as np

# 1. Cargar imagen de prueba
# En Google Colab o local, cargar en escala de grises o convertir a RGB
img = cv2.imread('mapa_muestra.png', cv2.IMREAD_GRAYSCALE)

# Si la imagen no se encuentra, generamos una sintética para probar
if img is None:
    img = np.zeros((100, 100), dtype=np.uint8)
    cv2.putText(img, 'R', (20, 70), cv2.FONT_HERSHEY_SIMPLEX, 2, 255, 3)

# 2. Definir nuevo ancho (width) y alto (height) escalados a 3x
# Nota: img.shape[0] es la altura (filas), img.shape[1] es el ancho (columnas)
height = int(img.shape[0] * 3)
width = int(img.shape[1] * 3)
nuevo_tamano = (width, height)  # cv2.resize espera (ancho, alto)

# 3. Redimensionamiento con diferentes métodos de interpolación
img_nearest = cv2.resize(img, nuevo_tamano, interpolation=cv2.INTER_NEAREST)
img_linear  = cv2.resize(img, nuevo_tamano, interpolation=cv2.INTER_LINEAR)
img_cubic   = cv2.resize(img, nuevo_tamano, interpolation=cv2.INTER_CUBIC)

# 4. Métodos con alineación geométrica exacta
img_nearest_exact = cv2.resize(img, nuevo_tamano, interpolation=cv2.INTER_NEAREST_EXACT)
img_linear_exact  = cv2.resize(img, nuevo_tamano, interpolation=cv2.INTER_LINEAR_EXACT)

# 5. Parámetro de zoom (área de recorte en píxeles)
zoom = 150

# 6. Visualización comparativa de los métodos principales con Zoom
fig, axis = plt.subplots(1, 3, figsize=(16, 6))

axis[0].imshow(img_nearest[0:zoom, 0:zoom], cmap='gray')
axis[0].set_title("Nearest Interpolation - ZOOM")
axis[0].axis("off")

axis[1].imshow(img_linear[0:zoom, 0:zoom], cmap='gray')
axis[1].set_title("Linear Interpolation - ZOOM")
axis[1].axis("off")

axis[2].imshow(img_cubic[0:zoom, 0:zoom], cmap='gray')
axis[2].set_title("Cubic Interpolation - ZOOM")
axis[2].axis("off")

plt.tight_layout()
plt.show()

# 7. Comparación adicional incluyendo las variantes EXACT
fig2, axis2 = plt.subplots(2, 3, figsize=(16, 10))

axis2[0, 0].imshow(img_nearest[0:zoom, 0:zoom], cmap='gray')
axis2[0, 0].set_title("Nearest Interpolation - ZOOM")
axis2[0, 0].axis("off")

axis2[0, 1].imshow(img_linear[0:zoom, 0:zoom], cmap='gray')
axis2[0, 1].set_title("Linear Interpolation - ZOOM")
axis2[0, 1].axis("off")

axis2[0, 2].imshow(img_cubic[0:zoom, 0:zoom], cmap='gray')
axis2[0, 2].set_title("Cubic Interpolation - ZOOM")
axis2[0, 2].axis("off")

axis2[1, 0].imshow(img_nearest_exact[0:zoom, 0:zoom], cmap='gray')
axis2[1, 0].set_title("Nearest Exact Interpolation - ZOOM")
axis2[1, 0].axis("off")

axis2[1, 1].imshow(img_linear_exact[0:zoom, 0:zoom], cmap='gray')
axis2[1, 1].set_title("Linear Exact Interpolation - ZOOM")
axis2[1, 1].axis("off")

# Desactivar eje sobrante
axis2[1, 2].axis("off")

plt.tight_layout()
plt.show()
```

## 🔍 Explicación paso a paso

### 1. Dimensiones y escalado
En NumPy, una matriz de imagen tiene dimensiones $(H, W) = (\text{filas}, \text{columnas})$:
- `img.shape[0]`: Altura ($H$).
- `img.shape[1]`: Ancho ($W$).

Multiplicar ambas por 3 genera una malla de destino con 9 veces más píxeles ($3 \times 3$).

> [!important] Orden de parámetros en OpenCV
> En `cv2.resize(src, dsize, ...)` el parámetro `dsize` recibe la tupla en orden **`(width, height)`**, es decir `(columnas, filas)`. Invertir este orden deformará la relación de aspecto de la imagen.

### 2. Comparación visual de métodos estándar

![[comparacion-interpolacion-zoom.png]]

- **Nearest Neighbor (`INTER_NEAREST`)**: Duplica los valores de los píxeles más próximos. Conserva bordes duros pero crea un efecto evidente de **bloques y escaleras (pixelación)**.
- **Bilinear (`INTER_LINEAR`)**: Calcula un promedio ponderado de los 4 vecinos más próximos. Suaviza la pixelación notablemente con muy bajo costo de procesamiento.
- **Bicubic (`INTER_CUBIC`)**: Ajusta polinomios cúbicos sobre una ventana de $4 \times 4$ vecinos (16 píxeles). Produce transiciones más suaves y bordes más limpios, con un costo computacional mayor.

### 3. Métodos exactos (`INTER_NEAREST_EXACT` e `INTER_LINEAR_EXACT`)

![[comparacion-interpolacion-exacta.png]]

- Las implementaciones estándar de OpenCV optimizan el cálculo para velocidad en procesadores vectoriales, introduciendo pequeños redondeos en el mapeo de coordenadas subpíxel.
- Las variantes **`_EXACT`** garantizan un cálculo matemáticamente exacto y simétrico de las coordenadas del píxel fuente:
  $$ x_{src} = (x_{dst} + 0.5) \times \frac{W_{src}}{W_{dst}} - 0.5 $$
- **¿Cuándo son fundamentales?**:
  - En **procesamiento volumétrico (imágenes médicas 3D, tomografías)** donde los errores de redondeo acumulados en sucesivos cortes alteran las mediciones anatómicas.
  - En pipelines de visión computacional y deep learning donde se requiere **reproducibilidad determinista exacta** entre distintas plataformas y arquitecturas de hardware.

## ✅ Resumen de métodos

| Método OpenCV | Vecinos considerados | Calidad visual | Rendimiento | Caso ideal de uso |
| :------------ | :------------------: | :------------: | :---------: | :---------------- |
| `INTER_NEAREST` | 1 ($1\times 1$) | Baja (pixelada) | Ultrarrápido | Segmentación con máscaras de clases / tiempo real |
| `INTER_LINEAR` | 4 ($2\times 2$) | Media (suave) | Rápido | Escalado general balanceado (predeterminado en OpenCV) |
| `INTER_CUBIC` | 16 ($4\times 4$) | Alta (muy suave) | Moderado | Agrandar fotos, preservación de curvas y texturas |
| `INTER_NEAREST_EXACT` | 1 | Exacta geométrica | Ultrarrápido | Datos volumétricos, máscaras exactas bit a bit |
| `INTER_LINEAR_EXACT` | 4 | Suave y reproducible | Rápido | Imágenes médicas 3D y simulaciones |

## ⚠️ Errores comunes

1. **Invertir dimensiones al llamar a `cv2.resize`**:
   - ❌ `cv2.resize(img, (height, width))`
   - ✅ `cv2.resize(img, (width, height))`
2. **Desbordamiento en el recorte de Zoom**:
   - Si `zoom > height` o `zoom > width`, el slice `[0:zoom, 0:zoom]` fallará o tomará dimensiones inesperadas. Asegurarse de que `zoom <= min(height, width)`.
3. **Imágenes a color en BGR vs RGB**:
   - `cv2.imread()` carga imágenes en orden **BGR**. Si se visualizan con `plt.imshow()`, deben convertirse primero con `cv2.cvtColor(img, cv2.COLOR_BGR2RGB)` para que los colores no se vean alterados.

## 🔗 Relacionado

- [[Interpolación]] — fundamentos teóricos y matemáticos
- [[Resolución de Imagen]] — muestreo espacial vs cuantización
- [[Tamaño de una Imagen Digital]] — cálculo de almacenamiento
- [[2026-08-18 - Interpolación y Redimensionamiento]] — clase correspondiente
- [[Inicio]] — mapa de contenidos

## 🏷️ Etiquetas

#visión-artificial #python #opencv #matplotlib #interpolación

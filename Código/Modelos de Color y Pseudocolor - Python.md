---
titulo: "Modelos de Color y Pseudocolor - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-11-03
clase: "[[2026-11-03 - Modelos de Color - RGB, CMYK, HSI y Pseudocolor]]"
tags:
  - visión-artificial
  - python
  - opencv
  - numpy
  - matplotlib
  - modelos-de-color
  - rgb
  - cmyk
  - hsi
  - hsv
  - pseudocolor
  - ecualización-en-color
aliases:
  - Código modelos de color Python
  - Código HSI HSV ecualización OpenCV
  - Código pseudocolor mapas térmicos Python
---

# Modelos de Color y Pseudocolor — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python (NumPy, OpenCV y Matplotlib)**:
> 1. Descomposición y visualización de una imagen en sus **canales RGB y CMYK**.
> 2. Conversión al espacio perceptual **HSI / HSV** con `cv2.cvtColor`.
> 3. Comparativa visual de la trampa clásica: **Ecualización directa en RGB** (que rompe los colores) vs **Ecualización en el canal de Intensidad ($V$/$I$) en HSV/HSI** (que preserva matiz y saturación).
> 4. Aplicación de técnicas de **Pseudocolor (Falso Color)** con `cv2.applyColorMap` (mapas `COLORMAP_JET`, `COLORMAP_INFERNO`, `COLORMAP_HOT`) para inspección de termografías y radiografías.

> [!info] Clases y conceptos relacionados
> 📅 Clase: [[2026-11-03 - Modelos de Color - RGB, CMYK, HSI y Pseudocolor]]  
> 🧠 Conceptos: [[Modelos de Color RGB y CMYK]], [[Espacio de Color HSI y Procesamiento Cromático]], [[Diagrama de Cromaticidad CIE y Pseudocolor]]

---

## 📦 Requisitos

```bash
pip install opencv-python numpy matplotlib
```

---

## 💻 Código Completo y Modular

```python
"""
Modelos de Color (RGB, CMYK, HSV/HSI) y Pseudocolor en Visión Artificial
Universidad de Nariño - Visión Artificial
"""

import cv2
import numpy as np
import matplotlib.pyplot as plt

# ==============================================================================
# 1. DESCOMPOSICIÓN DE CANALES RGB Y CMYK
# ==============================================================================
def descomponer_rgb_y_cmyk(imagen_rgb):
    """
    Descompone una imagen RGB en:
    - Canales primarios aditivos: R, G, B
    - Canales sustractivos: C, M, Y, K
    """
    # Canales aditivos
    R = imagen_rgb[:, :, 0]
    G = imagen_rgb[:, :, 1]
    B = imagen_rgb[:, :, 2]

    # Conversión a CMYK normalizado [0, 1]
    rgb_norm = imagen_rgb.astype(np.float32) / 255.0
    
    # K (Negro) es 1 - max(R, G, B)
    K = 1.0 - np.max(rgb_norm, axis=2)
    
    # Evitar divisiones por cero en píxeles totalmente negros
    denominador = 1.0 - K
    denominador[denominador == 0] = 1e-7

    C = (1.0 - rgb_norm[:, :, 0] - K) / denominador
    M = (1.0 - rgb_norm[:, :, 1] - K) / denominador
    Y = (1.0 - rgb_norm[:, :, 2] - K) / denominador

    # Escalar a uint8 [0, 255] para visualización
    cmyk = {
        "C": (np.clip(C, 0, 1) * 255).astype(np.uint8),
        "M": (np.clip(M, 0, 1) * 255).astype(np.uint8),
        "Y": (np.clip(Y, 0, 1) * 255).astype(np.uint8),
        "K": (np.clip(K, 0, 1) * 255).astype(np.uint8)
    }

    return (R, G, B), cmyk

# ==============================================================================
# 2. PROCESAMIENTO CROMÁTICO SEGURO: RGB vs HSV/HSI
# ==============================================================================
def comparar_ecualizacion_rgb_vs_hsv(imagen_rgb):
    """
    Demuestra por qué ecualizar directamente en RGB arruina el color,
    mientras que ecualizar la luminancia (canal V en HSV o I en HSI) es correcto.
    """
    # Método Erróneo: Ecualizar cada canal R, G y B por separado
    rgb_erroneo = np.zeros_like(imagen_rgb)
    for i in range(3):
        rgb_erroneo[:, :, i] = cv2.equalizeHist(imagen_rgb[:, :, i])

    # Método Correcto: Convertir a HSV, ecualizar SOLO el canal V (Valor / Intensidad)
    hsv = cv2.cvtColor(imagen_rgb, cv2.COLOR_RGB2HSV)
    hsv[:, :, 2] = cv2.equalizeHist(hsv[:, :, 2])
    rgb_correcto = cv2.cvtColor(hsv, cv2.COLOR_HSV2RGB)

    return rgb_erroneo, rgb_correcto

# ==============================================================================
# 3. APLICACIÓN DE PSEUDOCOLOR (MAPAS DE FALSO COLOR)
# ==============================================================================
def aplicar_pseudocolor(imagen_gris):
    """
    Aplica diferentes mapas de color predefinidos en OpenCV para resaltar
    detalles térmicos o radiológicos invisibles al ojo en niveles de gris.
    """
    colormaps = {
        "JET (Térmico clásico)": cv2.applyColorMap(imagen_gris, cv2.COLORMAP_JET),
        "INFERNO (Radiación continua)": cv2.applyColorMap(imagen_gris, cv2.COLORMAP_INFERNO),
        "HOT (Temperatura extrema)": cv2.applyColorMap(imagen_gris, cv2.COLORMAP_HOT)
    }
    # Convertir BGR de OpenCV a RGB para Matplotlib
    colormaps_rgb = {k: cv2.cvtColor(v, cv2.COLOR_BGR2RGB) for k, v in colormaps.items()}
    return colormaps_rgb

# ==============================================================================
# 4. SCRIPT PRINCIPAL DE DEMOSTRACIÓN
# ==============================================================================
if __name__ == "__main__":
    # 1. Crear una escena sintética de prueba con bajo contraste y colores variados
    H, W = 160, 200
    img_rgb = np.zeros((H, W, 3), dtype=np.uint8)
    img_rgb[:80, :100] = [180, 90, 80]    # Región rojiza
    img_rgb[:80, 100:] = [80, 160, 90]    # Región verdosa
    img_rgb[80:, :100] = [70, 90, 180]    # Región azulada
    img_rgb[80:, 100:] = [180, 170, 80]   # Región amarillenta
    
    # Comprimir rango de intensidades para simular imagen lavada o con baja luz
    img_baja_luz = (img_rgb * 0.55).astype(np.uint8)

    # 2. Descomposición
    (R, G, B), cmyk = descomponer_rgb_y_cmyk(img_baja_luz)

    # 3. Comparación de ecualización
    erroneo_rgb, correcto_hsv = comparar_ecualizacion_rgb_vs_hsv(img_baja_luz)

    # 4. Crear imagen monocromática con gradiente sutil y hotspots térmicos
    y, x = np.mgrid[0:H, 0:W]
    fondo = 70 + 20 * np.sin(x / 25)
    foco_calor = 90 * np.exp(-((x - 100)**2 + (y - 80)**2) / (2 * 20**2))
    img_termica_gris = np.clip(fondo + foco_calor, 0, 255).astype(np.uint8)

    # 5. Pseudocolor
    pseudo_maps = aplicar_pseudocolor(img_termica_gris)

    # --- VISUALIZACIÓN ---
    plt.figure(figsize=(15, 8))

    plt.subplot(2, 3, 1)
    plt.imshow(img_baja_luz)
    plt.title("1. Original Bajo Contraste", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 3, 2)
    plt.imshow(erroneo_rgb)
    plt.title("2. Error: Ecualizar RGB\n(Colores distorsionados)", color="red", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 3, 3)
    plt.imshow(correcto_hsv)
    plt.title("3. Correcto: Ecualizar en HSV/HSI\n(Canal V ecualizado / Matiz intacto)", color="green", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 3, 4)
    plt.imshow(img_termica_gris, cmap="gray")
    plt.title("4. Termografía en Grises\n(Apenas distinguible por el ojo)", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 3, 5)
    plt.imshow(pseudo_maps["JET (Térmico clásico)"])
    plt.title("5. Pseudocolor JET\n(Punto crítico en rojo fuego)", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 3, 6)
    plt.imshow(pseudo_maps["INFERNO (Radiación continua)"])
    plt.title("6. Pseudocolor INFERNO\n(Gradiente continuo de radiación)", fontweight="bold")
    plt.axis("off")

    plt.tight_layout()
    plt.show()
    print("[*] Demostración de modelos de color y pseudocolor completada exitosamente.")
```

---

## 🔍 Explicación Paso a Paso

### 1. Descomposición a CMYK
En la industria gráfica, cada canal representa el porcentaje de cobertura de tinta. El canal clave $K$ extrae el componente acromático compartido, reduciendo el gasto de tintas $C, M$ e $Y$.

### 2. Por qué OpenCV usa HSV en lugar de HSI
OpenCV incluye de forma nativa la conversión `cv2.COLOR_BGR2HSV` y `cv2.COLOR_RGB2HSV`. El espacio HSV (*Hue, Saturation, Value*) es una variante poliédrica muy próxima al HSI clásico:
- El canal $V$ (*Value*) corresponde al máximo entre los canales ($\max(R, G, B)$), mientras que en HSI el canal $I$ es el promedio ($\frac{R+G+B}{3}$).
- En ambos modelos, la separación entre crominancia $(H, S)$ y luminancia $(V$ o $I)$ es idéntica, permitiendo procesar el contraste sin distorsionar el tono.

### 3. Pseudocolor mediante LUTs
La función `cv2.applyColorMap(src, colormap)` toma cada valor escalar $g \in [0, 255]$ y lo indexa en una tabla de colores pre-calculada de $256 \times 3$, transformando sutiles gradientes monocromáticos en transiciones cromáticas perceptualmente evidentes.

---

## ⚠️ Errores Comunes y Mejores Prácticas

| Error Común | Causa Raíz | Solución |
| :--- | :--- | :--- |
| **Colores invertidos (Azul en vez de Rojo)** | Confusión entre el orden **BGR** nativo de OpenCV y el orden **RGB** de Matplotlib. | Convertir siempre con `cv2.cvtColor(img, cv2.COLOR_BGR2RGB)` antes de graficar con `plt.imshow`. |
| **Rango de Matiz ($H$) desbordado** | En OpenCV, el matiz $H$ se almacena en el rango $[0, 180]$ (para caber en un `uint8`), no en $[0, 360]$. | Al buscar un color específico (ej. verde a $120^\circ$), buscar el valor dividido por dos ($60$). |
| **Resta CMY negativa** | Restar directamente con tipos `np.uint8` produce subdesbordamiento modular. | Convertir a `np.float32` normalizado $[0, 1]$ antes de calcular $C, M, Y, K$. |

---

## 🔗 Notas Relacionadas
- [[2026-11-03 - Modelos de Color - RGB, CMYK, HSI y Pseudocolor]] — Clase teórica detallada.
- [[Modelos de Color RGB y CMYK]] — Fundamentos de síntesis aditiva y sustractiva.
- [[Espacio de Color HSI y Procesamiento Cromático]] — Desacoplamiento de componentes.
- [[Diagrama de Cromaticidad CIE y Pseudocolor]] — Locus espectral y mapas térmicos.
- [[Histogramas y Ecualización de Imagen]] — Algoritmos de contraste.
- [[Inicio]] — Portada principal de la bóveda.

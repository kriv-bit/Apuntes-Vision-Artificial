---
titulo: "Morfología Matemática y Hit-or-Miss - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-11-03
clase: "[[2026-11-03 - Morfología Matemática - Binaria y Escala de Grises]]"
tags:
  - visión-artificial
  - python
  - opencv
  - numpy
  - matplotlib
  - morfología-matemática
  - imágenes-binarias
  - escala-de-grises
  - erosión
  - dilatación
  - apertura
  - cierre
  - hit-or-miss
  - extracción-de-bordes
  - adelgazamiento
  - esqueletización
aliases:
  - Código morfología matemática Python
  - Código erosión dilatación apertura cierre OpenCV
  - Código Hit-or-Miss Python
  - Código esqueletización y gradiente morfológico
---

# Morfología Matemática, Esqueletización y Escala de Grises — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python (NumPy, OpenCV y Matplotlib)**:
> 1. Creación de **elementos estructurantes** (`MORPH_RECT`, `MORPH_CROSS`, `MORPH_ELLIPSE`) con `cv2.getStructuringElement`.
> 2. Operaciones morfológicas binarias fundamentales: **Erosión** (`cv2.erode`) y **Dilatación** (`cv2.dilate`).
> 3. Verificación computacional del **Principio de Dualidad**: $(A \ominus B)^c = A^c \oplus \hat{B}$.
> 4. **Extracción morfológica de bordes**: $\text{Borde} = \text{Original} - \text{Erosión}$.
> 5. **Transformación Hit-or-Miss** (`MORPH_HITMISS`) con kernels tri-estado para detección de esquinas y banco de kernels rotados.
> 6. **Adelgazamiento y Esqueletización Morfológica**: reducción a 1 píxel preservando topología (casos reales: huellas con sobrepresión y caracteres de placas vehiculares en foto-multas).
> 7. **Morfología en Escala de Grises**: Erosión (mínimo local), Dilatación (máximo local), Gradiente Morfológico (`MORPH_GRADIENT`) y filtros Top-Hat / Black-Hat.

> [!info] Clases y conceptos relacionados
> 📅 Clase: [[2026-11-03 - Morfología Matemática - Binaria y Escala de Grises]]  
> 🧠 Conceptos: [[Operaciones Morfológicas Binarias]], [[Transformación Hit-or-Miss]], [[Adelgazamiento, Espesamiento y Esqueletización]], [[Morfología en Escala de Grises]]

---

## 📦 Requisitos

```bash
pip install opencv-python numpy matplotlib
```

---

## 💻 Código Completo y Modular

```python
"""
Morfología Matemática: Binaria, Esqueletización y Escala de Grises
Universidad de Nariño - Visión Artificial
"""

import cv2
import numpy as np
import matplotlib.pyplot as plt

# ==============================================================================
# 1. ELEMENTOS ESTRUCTURANTES (SE)
# ==============================================================================
def obtener_elementos_estructurantes(ksize=5):
    """
    Retorna los 3 tipos estándar de SE en OpenCV:
    - Rectangular (vecindad de 8 conectividad)
    - Cruz (vecindad de 4 conectividad)
    - Elipse (aproximación isotrópica)
    """
    return {
        "Rectangular": cv2.getStructuringElement(cv2.MORPH_RECT, (ksize, ksize)),
        "Cruz": cv2.getStructuringElement(cv2.MORPH_CROSS, (ksize, ksize)),
        "Elipse": cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (ksize, ksize))
    }

# ==============================================================================
# 2. DUALIDAD MORFOLÓGICA Y EXTRACCIÓN DE BORDES
# ==============================================================================
def verificar_dualidad(imagen_binaria, kernel):
    """
    Demuestra computacionalmente: (A ⊖ B)^c == A^c ⊕ B_reflejado
    """
    erosion_directa = cv2.erode(imagen_binaria, kernel)
    
    # Dualidad: fondo -> dilatar -> complementar
    fondo = cv2.bitwise_not(imagen_binaria)
    kernel_reflejado = cv2.flip(kernel, -1)
    fondo_dilatado = cv2.dilate(fondo, kernel_reflejado)
    erosion_por_dualidad = cv2.bitwise_not(fondo_dilatado)
    
    diff = np.max(np.abs(erosion_directa.astype(int) - erosion_por_dualidad.astype(int)))
    print(f"[*] Comprobación de dualidad: Diferencia máxima = {diff} píxeles.")
    return erosion_directa, erosion_por_dualidad

def extraer_borde_morfológico(imagen_binaria, ksize=3):
    """
    Borde Interno = Original - Erosión
    """
    kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (ksize, ksize))
    erosion = cv2.erode(imagen_binaria, kernel)
    borde = cv2.subtract(imagen_binaria, erosion)
    return borde

# ==============================================================================
# 3. TRANSFORMACIÓN HIT-OR-MISS (TODO O NADA)
# ==============================================================================
def hit_or_miss_esquinas(imagen_binaria):
    """
    Detección de esquinas usando matrices tri-estado:
      +1: Coincidencia con Objeto (Hit)
      -1: Coincidencia con Fondo (Miss)
       0: Indiferente (Don't care)
    """
    k_esq_sd = np.array([
        [ 0, -1, -1],
        [ 1,  1, -1],
        [ 0,  1,  0]
    ], dtype=np.int32)

    k_esq_si = np.array([
        [-1, -1,  0],
        [-1,  1,  1],
        [ 0,  1,  0]
    ], dtype=np.int32)

    k_esq_id = np.array([
        [ 0,  1,  0],
        [ 1,  1, -1],
        [ 0, -1, -1]
    ], dtype=np.int32)

    k_esq_ii = np.array([
        [ 0,  1,  0],
        [-1,  1,  1],
        [-1, -1,  0]
    ], dtype=np.int32)

    resultado = np.zeros_like(imagen_binaria)
    for k in [k_esq_sd, k_esq_si, k_esq_id, k_esq_ii]:
        acierto = cv2.morphologyEx(imagen_binaria, cv2.MORPH_HITMISS, k)
        resultado = cv2.bitwise_or(resultado, acierto)

    return resultado

# ==============================================================================
# 4. ADELGAZAMIENTO Y ESQUELETIZACIÓN TOPOLÓGICA (HUELLAS Y MATRÍCULAS)
# ==============================================================================
def esqueletizacion_morfologica(imagen_binaria):
    """
    Reduce el objeto a un esqueleto de 1 píxel preservando conectividad.
    Útil para:
    - Huellas dactilares aplastadas (elimina sombra fea por sobrepresión).
    - Reconocimiento de caracteres en placas vehiculares (OCR invulnerable al grosor).
    """
    skel = np.zeros(imagen_binaria.shape, np.uint8)
    element = cv2.getStructuringElement(cv2.MORPH_CROSS, (3, 3))
    temp_img = imagen_binaria.copy()

    while True:
        eroded = cv2.erode(temp_img, element)
        temp = cv2.dilate(eroded, element)
        temp = cv2.subtract(temp_img, temp)
        skel = cv2.bitwise_or(skel, temp)
        temp_img = eroded.copy()
        if cv2.countNonZero(temp_img) == 0:
            break

    return skel

# ==============================================================================
# 5. MORFOLOGÍA EN ESCALA DE GRISES (MODELO TOPOGRÁFICO)
# ==============================================================================
def morfologia_escala_grises(imagen_gris, ksize=5):
    """
    Operaciones morfológicas en grises:
    - Dilatación = Máximo Local (eleva valles y expande zonas claras)
    - Erosión = Mínimo Local (hunde cumbres y expande zonas oscuras)
    - Gradiente Morfológico = Máximo Local - Mínimo Local
    - Top-Hat = Original - Apertura (aísla picos brillantes sobre fondo oscuro)
    - Black-Hat = Cierre - Original (aísla depresiones oscuras)
    """
    kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (ksize, ksize))
    
    dilatacion = cv2.dilate(imagen_gris, kernel)
    erosion = cv2.erode(imagen_gris, kernel)
    gradiente = cv2.morphologyEx(imagen_gris, cv2.MORPH_GRADIENT, kernel)
    tophat = cv2.morphologyEx(imagen_gris, cv2.MORPH_TOPHAT, kernel)
    blackhat = cv2.morphologyEx(imagen_gris, cv2.MORPH_BLACKHAT, kernel)
    
    return {
        "dilatacion": dilatacion,
        "erosion": erosion,
        "gradiente": gradiente,
        "tophat": tophat,
        "blackhat": blackhat
    }

# ==============================================================================
# 6. DEMOSTRACIÓN PRINCIPAL Y VISUALIZACIÓN
# ==============================================================================
if __name__ == "__main__":
    # --- PARTE A: DOMINIO BINARIO ---
    img_bin = np.zeros((140, 140), dtype=np.uint8)
    cv2.putText(img_bin, "A", (25, 115), cv2.FONT_HERSHEY_SIMPLEX, 4.2, 255, 16, cv2.LINE_AA)
    _, img_bin = cv2.threshold(img_bin, 127, 255, cv2.THRESH_BINARY)

    # 1. Borde Morfológico (Original - Erosión)
    borde = extraer_borde_morfológico(img_bin, ksize=5)

    # 2. Esqueletización (Thinning topológico)
    esqueleto = esqueletizacion_morfologica(img_bin)

    # 3. Hit-or-Miss
    esquinas = hit_or_miss_esquinas(img_bin)

    # --- PARTE B: DOMINIO ESCALA DE GRISES ---
    # Superficie con iluminación variable
    x = np.linspace(-3, 3, 140)
    y = np.linspace(-3, 3, 140)
    X, Y = np.meshgrid(x, y)
    fondo_degradado = (100 + 40 * np.sin(X) + 40 * np.cos(Y)).astype(np.uint8)
    # Agregar figura con ruido
    img_gris = cv2.add(fondo_degradado, (img_bin * 0.4).astype(np.uint8))
    
    res_gris = morfologia_escala_grises(img_gris, ksize=5)

    # --- VISUALIZACIÓN COMPLETA ---
    plt.figure(figsize=(16, 7))

    plt.subplot(2, 4, 1)
    plt.imshow(img_bin, cmap="gray")
    plt.title("1. Binaria Gruesa (Letra 'A')", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 4, 2)
    plt.imshow(borde, cmap="gray")
    plt.title("2. Borde (Original - Erosión)", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 4, 3)
    plt.imshow(esqueleto, cmap="gray")
    plt.title("3. Esqueleto (Adelgazamiento 1 px)", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 4, 4)
    overlay_hm = cv2.cvtColor(img_bin, cv2.COLOR_GRAY2RGB)
    pts = np.argwhere(esquinas > 0)
    for py, px in pts:
        cv2.circle(overlay_hm, (px, py), 3, (255, 0, 0), -1)
    plt.imshow(overlay_hm)
    plt.title("4. Hit-or-Miss (Esquinas en rojo)", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 4, 5)
    plt.imshow(img_gris, cmap="gray")
    plt.title("5. Imagen Grises (Topografía)", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 4, 6)
    plt.imshow(res_gris["gradiente"], cmap="gray")
    plt.title("6. Gradiente Grises (Máx - Mín)", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 4, 7)
    plt.imshow(res_gris["tophat"], cmap="gray")
    plt.title("7. White Top-Hat (Detalles Clari)", fontweight="bold")
    plt.axis("off")

    plt.subplot(2, 4, 8)
    plt.imshow(res_gris["blackhat"], cmap="gray")
    plt.title("8. Black-Hat (Detalles Oscuros)", fontweight="bold")
    plt.axis("off")

    plt.tight_layout()
    plt.show()
    print("[*] Ejecución completada con éxito.")
```

---

## 🔍 Explicación Paso a Paso

### 1. Extracción de Bordes (`Original - Erosión`)
Al sustraer la imagen erosionada a la original, los píxeles interiores compartidos se anulan (`255 - 255 = 0`). Los únicos píxeles supervivientes (`255 - 0 = 255`) corresponden a la capa más externa del contorno desbastada por la sonda morfológica.

### 2. Esqueletización y Mitigación de Sombras
El algoritmo iterativo extrae la esencia topológica restando en cada ciclo las zonas dilatadas de la erosión. De este modo, si un usuario genera una masa gruesa o mancha negra en un sensor de huella digital por sobrepresión física, el esqueleto final colapsa al eje medio axial unívoco de las crestas papilares.

### 3. Máximo y Mínimo Local en Grises
- `cv2.dilate` actúa como un **filtro de máximo no lineal**: para cada píxel, toma el valor más alto en el vecindario del kernel. En una superficie topográfica, esto expande las cimas y aplana hacia arriba los valles estrechos.
- `cv2.erode` actúa como un **filtro de mínimo no lineal**: toma el valor más bajo, expandiendo los valles oscuros y rebajando la altura de las cimas brillantes.
- El **Gradiente Morfológico** (`cv2.morphologyEx(..., cv2.MORPH_GRADIENT)`) computa la diferencia `Máximo - Mínimo`, realzando transiciones abruptas de pendiente sin importar la dirección del borde.

---

## ⚠️ Errores Comunes y Mejores Prácticas

| Error Común | Causa Raíz | Solución |
| :--- | :--- | :--- |
| **Borde vacío (todo negro)** | El elemento estructurante es demasiado grande y la erosión eliminó todo el objeto a 0. | Usar elementos pequeños ($3\times 3$) para bordes finos de 1 píxel. |
| **Hit-or-Miss arroja ruido o falla** | Kernel definido con `dtype=np.uint8` en vez de `np.int32`. | Usar siempre `np.int32` para preservar valores `-1` (Miss). |
| **Confusión entre Top-Hat y Gradiente** | El gradiente resta `Dilatación - Erosión`; el Top-Hat resta `Original - Apertura`. | Usar Gradiente para contornos y Top-Hat para aislar motas sobre fondos desparejos. |
| **Desbordamiento numérico en resta manual** | Restar directamente arrays uint8 con `-` (provoca subdesbordamiento modular en NumPy). | Usar siempre `cv2.subtract(A, B)` que satura a cero. |

---

## 🔗 Notas Relacionadas
- [[2026-11-03 - Morfología Matemática - Binaria y Escala de Grises]] — Clase teórica detallada.
- [[Operaciones Morfológicas Binarias]] — Fundamentos de teoría de conjuntos y dualidad.
- [[Transformación Hit-or-Miss]] — Detección de patrones geométricos y minucias.
- [[Adelgazamiento, Espesamiento y Esqueletización]] — Preservación de homotopía y casos prácticos.
- [[Morfología en Escala de Grises]] — Modelo topográfico 3D y operadores no lineales.
- [[Inicio]] — Hub principal de la asignatura.

---
titulo: "Morfología Matemática y Hit-or-Miss - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-11-03
clase: "[[2026-11-03 - Morfología Matemática en Imágenes Binarias]]"
tags:
  - visión-artificial
  - python
  - opencv
  - numpy
  - matplotlib
  - morfología-matemática
  - imágenes-binarias
  - erosión
  - dilatación
  - apertura
  - cierre
  - hit-or-miss
aliases:
  - Código morfología matemática Python
  - Código erosión dilatación apertura cierre OpenCV
  - Código Hit-or-Miss Python
---

# Morfología Matemática y Hit-or-Miss — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python (NumPy, OpenCV y Matplotlib)**:
> 1. Creación de **elementos estructurantes** rectangulares, en cruz y elípticos con `cv2.getStructuringElement`.
> 2. Operaciones morfológicas fundamentales: **Erosión** (`cv2.erode`) y **Dilatación** (`cv2.dilate`).
> 3. Verificación computacional exacta del **Principio de Dualidad Morfológica**: $(A \ominus B)^c = A^c \oplus \hat{B}$.
> 4. Operaciones compuestas: **Apertura** (`MORPH_OPEN`) y **Cierre** (`MORPH_CLOSE`) para eliminación de ruido exterior y sellado de fracturas.
> 5. **Transformación Hit-or-Miss** (`MORPH_HITMISS`) con kernels tri-estado para detección de esquinas y patrones.
> 6. Banco de kernels rotados para lograr **invarianza a rotación** en detección de minucias biométricas (tipo huella dactilar).

> [!info] Clases y conceptos relacionados
> 📅 Clase: [[2026-11-03 - Morfología Matemática en Imágenes Binarias]]  
> 🧠 Conceptos: [[Operaciones Morfológicas Binarias]] y [[Transformación Hit-or-Miss]]

---

## 📦 Requisitos

```bash
pip install opencv-python numpy matplotlib
```

---

## 💻 Código Completo y Modular

```python
"""
Morfología Matemática en Imágenes Binarias y Transformación Hit-or-Miss
Universidad de Nariño - Visión Artificial
"""

import cv2
import numpy as np
import matplotlib.pyplot as plt

# ==============================================================================
# 1. GENERACIÓN DE ELEMENTOS ESTRUCTURANTES (SE)
# ==============================================================================
def obtener_elementos_estructurantes(ksize=5):
    """
    Retorna un diccionario con los 3 tipos estándar de SE en OpenCV:
    - Rectangular (vecindad de 8 conectividad extendida)
    - Cruz (vecindad de 4 conectividad extendida)
    - Elipse (aproximación discreta de disco isotrópico)
    """
    return {
        "Rectangular": cv2.getStructuringElement(cv2.MORPH_RECT, (ksize, ksize)),
        "Cruz": cv2.getStructuringElement(cv2.MORPH_CROSS, (ksize, ksize)),
        "Elipse": cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (ksize, ksize))
    }

# ==============================================================================
# 2. VERIFICACIÓN DEL TEOREMA DE DUALIDAD MORFOLÓGICA
# ==============================================================================
def verificar_dualidad(imagen_binaria, kernel):
    """
    Demuestra computacionalmente que:
      (A ⊖ B)^c == A^c ⊕ B_reflejado
    Retorna la diferencia absoluta máxima (debe ser 0).
    """
    # Camino 1: Erosión directa del objeto
    erosion_directa = cv2.erode(imagen_binaria, kernel)
    
    # Camino 2: Dualidad a través del fondo
    # Complementar A -> Dilatar fondo con kernel reflejado -> Complementar resultado
    fondo = cv2.bitwise_not(imagen_binaria)
    kernel_reflejado = cv2.flip(kernel, -1)  # Reflejar horizontal y verticalmente
    fondo_dilatado = cv2.dilate(fondo, kernel_reflejado)
    erosion_por_dualidad = cv2.bitwise_not(fondo_dilatado)
    
    # Comprobación de equivalencia matemática estricta
    diferencia = np.max(np.abs(erosion_directa.astype(int) - erosion_por_dualidad.astype(int)))
    print(f"[*] Comprobación de dualidad: Diferencia máxima = {diferencia} píxeles.")
    return erosion_directa, erosion_por_dualidad, diferencia

# ==============================================================================
# 3. FILTRADO POR APERTURA Y CIERRE
# ==============================================================================
def filtrar_apertura_y_cierre(imagen_ruidosa, ksize=7):
    """
    Aplica:
    - Apertura (MORPH_OPEN): elimina islas de ruido exterior y rompe istmos.
    - Cierre (MORPH_CLOSE): sella fracturas internas y rellena huecos.
    """
    kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (ksize, ksize))
    apertura = cv2.morphologyEx(imagen_ruidosa, cv2.MORPH_OPEN, kernel)
    cierre = cv2.morphologyEx(imagen_ruidosa, cv2.MORPH_CLOSE, kernel)
    
    # Filtro morfológico secuencial completo (Apertura seguida de Cierre)
    filtrado_completo = cv2.morphologyEx(apertura, cv2.MORPH_CLOSE, kernel)
    return apertura, cierre, filtrado_completo

# ==============================================================================
# 4. TRANSFORMACIÓN HIT-OR-MISS (TODO O NADA)
# ==============================================================================
def hit_or_miss_esquinas(imagen_binaria):
    """
    Detecta esquinas mediante un banco de kernels Hit-or-Miss.
    Matriz tri-estado:
      +1: Coincidencia con Objeto (Hit)
      -1: Coincidencia con Fondo (Miss)
       0: Indiferente (Don't care)
    """
    # Kernel para esquina superior derecha
    k_esq_sd = np.array([
        [ 0, -1, -1],
        [ 1,  1, -1],
        [ 0,  1,  0]
    ], dtype=np.int32)

    # Kernel para esquina superior izquierda
    k_esq_si = np.array([
        [-1, -1,  0],
        [-1,  1,  1],
        [ 0,  1,  0]
    ], dtype=np.int32)

    # Kernel para esquina inferior derecha
    k_esq_id = np.array([
        [ 0,  1,  0],
        [ 1,  1, -1],
        [ 0, -1, -1]
    ], dtype=np.int32)

    # Kernel para esquina inferior izquierda
    k_esq_ii = np.array([
        [ 0,  1,  0],
        [-1,  1,  1],
        [-1, -1,  0]
    ], dtype=np.int32)

    kernels = [k_esq_sd, k_esq_si, k_esq_id, k_esq_ii]
    resultado_esquinas = np.zeros_like(imagen_binaria)

    for k in kernels:
        acierto = cv2.morphologyEx(imagen_binaria, cv2.MORPH_HITMISS, k)
        resultado_esquinas = cv2.bitwise_or(resultado_esquinas, acierto)

    return resultado_esquinas

# ==============================================================================
# 5. DEMOSTRACIÓN DE INVARIANZA A ROTACIÓN (HUELLA DACTILAR / MINUCIAS)
# ==============================================================================
def detectar_terminaciones_invariante_rotacion(esqueleto):
    """
    Detecta terminaciones de línea (ridge endings) en un esqueleto morfológico,
    rotando el kernel base en 8 orientaciones angulares (pasos de 45°).
    """
    # Kernel base para terminación orientada hacia arriba
    # (El píxel central tiene un vecino abajo y está rodeado de fondo arriba y a los lados)
    k_base = np.array([
        [-1, -1, -1],
        [-1,  1, -1],
        [ 0,  1,  0]
    ], dtype=np.int32)

    resultado = np.zeros_like(esqueleto)
    
    # Banco de 4 orientaciones ortogonales (90°, 180°, 270°) y diagonales
    # Generamos rotaciones usando rot90 de numpy
    for r in range(4):
        k_rot = np.rot90(k_base, r)
        det = cv2.morphologyEx(esqueleto, cv2.MORPH_HITMISS, k_rot)
        resultado = cv2.bitwise_or(resultado, det)

    # Kernel base diagonal
    k_diag = np.array([
        [-1, -1, -1],
        [-1,  1,  0],
        [-1,  0,  1]
    ], dtype=np.int32)
    
    for r in range(4):
        k_rot = np.rot90(k_diag, r)
        det = cv2.morphologyEx(esqueleto, cv2.MORPH_HITMISS, k_rot)
        resultado = cv2.bitwise_or(resultado, det)

    return resultado

# ==============================================================================
# 6. SCRIPT PRINCIPAL DE PRUEBA Y VISUALIZACIÓN
# ==============================================================================
if __name__ == "__main__":
    # 1. Crear imagen sintética de prueba con figuras, istmos y huecos
    img_test = np.zeros((150, 150), dtype=np.uint8)
    # Rectángulo grande
    img_test[30:120, 30:70] = 255
    # Cuadrado pequeño unido por un istmo delgado de 2 píxeles
    img_test[50:100, 90:130] = 255
    img_test[74:76, 70:90] = 255
    # Hueco interno en el rectángulo
    img_test[60:80, 45:55] = 0
    # Ruido externo puntual
    img_test[15, 15] = 255
    img_test[130, 20] = 255

    # 2. Comprobar dualidad morfológica
    k_test = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))
    er_dir, er_dual, diff = verificar_dualidad(img_test, k_test)

    # 3. Aplicar apertura y cierre
    ap, ci, total = filtrar_apertura_y_cierre(img_test, ksize=5)

    # 4. Hit-or-Miss de esquinas
    esquinas = hit_or_miss_esquinas(img_test)

    # 5. Visualización
    plt.figure(figsize=(15, 6))

    plt.subplot(1, 4, 1)
    plt.imshow(img_test, cmap="gray")
    plt.title("Original (con istmo, agujero e islas)")
    plt.axis("off")

    plt.subplot(1, 4, 2)
    plt.imshow(ap, cmap="gray")
    plt.title("Apertura (Rompe istmo y quita islas)")
    plt.axis("off")

    plt.subplot(1, 4, 3)
    plt.imshow(ci, cmap="gray")
    plt.title("Cierre (Sella agujero interior)")
    plt.axis("off")

    plt.subplot(1, 4, 4)
    # Superponer esquinas en rojo sobre la imagen
    overlay = cv2.cvtColor(img_test, cv2.COLOR_GRAY2RGB)
    pts = np.argwhere(esquinas > 0)
    for y, x in pts:
        cv2.circle(overlay, (x, y), 3, (255, 0, 0), -1)
    plt.imshow(overlay)
    plt.title(f"Hit-or-Miss ({len(pts)} esquinas detectadas)")
    plt.axis("off")

    plt.tight_layout()
    plt.show()
    print("[*] Proceso finalizado exitosamente.")
```

---

## 🔍 Explicación Paso a Paso

### 1. Elementos estructurantes en OpenCV (`cv2.getStructuringElement`)
- `cv2.MORPH_RECT`: Matriz llena de unos. Ideal para esquinas rectas y estructuras ortogonales.
- `cv2.MORPH_CROSS`: Matriz en cruz (vecinos arriba, abajo, izquierda, derecha). Adecuada para conectividad $N_4$.
- `cv2.MORPH_ELLIPSE`: Matriz que aproxima un círculo inscrito. Preserva mejor la curvatura natural de los objetos sin inducir distorsiones angulares.

### 2. Implementación de la Dualidad
El código demuestra que:
```python
erosion_por_dualidad = cv2.bitwise_not(cv2.dilate(cv2.bitwise_not(img), cv2.flip(k, -1)))
```
El resultado numérico confirma `diff == 0`, validando la ley fundamental: dilatar el fondo equivale a erosionar el objeto.

### 3. Apertura vs Cierre
- La función `cv2.morphologyEx(img, cv2.MORPH_OPEN, kernel)` ejecuta internamente `cv2.dilate(cv2.erode(img, kernel), kernel)`.
- La función `cv2.morphologyEx(img, cv2.MORPH_CLOSE, kernel)` ejecuta internamente `cv2.erode(cv2.dilate(img, kernel), kernel)`.

### 4. Hit-or-Miss y Valores Tri-Estado
- El tipo de dato del kernel debe ser explícitamente `np.int32` para admitir valores negativos `-1`.
- Si se deja por defecto en `np.uint8`, el `-1` se desborda a `255`, destruyendo la semántica de "Miss".

---

## ⚠️ Errores Comunes y Mejores Prácticas

| Error Común | Causa Raíz | Solución |
| :--- | :--- | :--- |
| **Hit-or-Miss no detecta nada o devuelve toda la pantalla blanca** | Se creó el kernel con `dtype=np.uint8` en lugar de `np.int32`. En uint8, `-1` se convierte en `255`. | Declarar siempre `dtype=np.int32` al definir kernels Hit-or-Miss. |
| **El objeto desaparece al aplicar erosión** | El elemento estructurante es más grande que el grosor mínimo del objeto. | Seleccionar tamaños de kernel impares pequeños ($3\times 3$ o $5\times 5$) acordes al espesor de la figura. |
| **Inversión de fondo/objeto** | Se asume que el objeto es negro y el fondo blanco. La convención morfológica estándar exige **objeto blanco (255) y fondo negro (0)**. | Usar `cv2.bitwise_not()` o umbralizar con `cv2.THRESH_BINARY_INV` para normalizar antes de operar. |
| **Asimetría en la dilatación con kernels no simétricos** | No reflejar el elemento estructurante ($\hat{B}$). | Si el kernel no es simétrico respecto al origen, reflejarlo con `cv2.flip(kernel, -1)`. |
| **Tamaño de kernel par** | Los tamaños pares carecen de un centro geométrico único e inequívoco, introduciendo desplazamientos de medio píxel. | Utilizar siempre dimensiones impares ($3\times 3$, $5\times 5$, $7\times 7$, etc.). |

---

## 🔗 Notas Relacionadas
- [[2026-11-03 - Morfología Matemática en Imágenes Binarias]] — Apuntes completos de la sesión.
- [[Operaciones Morfológicas Binarias]] — Teoría y propiedades matemáticas.
- [[Transformación Hit-or-Miss]] — Detección de minucias y biometría.
- [[Inicio]] — Portada principal del repositorio.

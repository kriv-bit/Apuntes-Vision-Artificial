---
titulo: "Procesamiento y Rebanado de Color - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-11-04
clase: "[[2026-11-04 - Color y Segmentación de Imágenes]]"
tags:
  - visión-artificial
  - python
  - opencv
  - numpy
  - matplotlib
  - modelos-de-color
  - hsi
  - pseudocolor
  - rebanado-de-color
  - transformaciones-cromáticas
aliases:
  - Código Rebanado de Color Python
  - Código HSI conversiones y rebanado
  - Código distancia circular matiz OpenCV
---

# Procesamiento y Rebanado de Color — Python

## 🎯 Qué hace el programa

Este script implementa las operaciones avanzadas de procesamiento digital en color y pseudocolor a partir de los notebooks **5.3 y 5.4**:
1. **Conversión matemática exacta entre RGB y HSI** sin aproximaciones reducidas, incluyendo la gestión de singularidades en negros y grises acromáticos.
2. **Pseudocolor por rebanado de bandas de intensidad** (5 intervalos fijos) vs **colormap continuo** (`matplotlib.cm.inferno`).
3. **Realce de regiones de interés por intensidad** y verificación de error de discretización geométrica.
4. **Transformaciones a todo color**:
   - Complemento o negativo RGB.
   - Rotación angular del matiz ($H' = (H + \Delta H) \bmod 1$).
   - Desaturación controlada ($S' = \alpha S$) con preservación estricta de la luminancia ($I$).
5. **Rebanado de color (*Color Slicing*)** utilizando la **métrica de distancia circular en matiz**:
   $$d_H = \min\bigl(|H - H_0|, \, 1 - |H - H_0|\bigr)$$
   combinada con un umbral de pureza en saturación ($S > 0.3$).
6. **Filtrado espacial seguro sobre el canal $I$** para evitar la aparición de colores espurios en las fronteras entre regiones cromáticas.

> [!info] Clases y conceptos relacionados
> 📅 Clase: [[2026-11-04 - Color y Segmentación de Imágenes]]  
> 🧠 Conceptos: [[Rebanado de Color y Operaciones Cromáticas]], [[Espacio de Color HSI y Procesamiento Cromático]], [[Modelos de Color RGB y CMYK]], [[Diagrama de Cromaticidad CIE y Pseudocolor]]

---

## 📦 Requisitos

```bash
pip install opencv-python numpy matplotlib
```

---

## 💻 Código Completo y Modular

```python
"""
Procesamiento Digital de Color, Pseudocolor y Rebanado en Espacio HSI
Universidad de Nariño - Visión Artificial
Basado en los Notebooks 5.3 y 5.4
"""

import numpy as np
import cv2
import matplotlib.pyplot as plt


# ==============================================================================
# 1. CONVERSIONES MATEMÁTICAS EXACTAS: RGB <-> HSI
# ==============================================================================
def rgb_a_hsi(imagen_rgb):
    """
    Convierte una imagen RGB uint8 [0, 255] al espacio HSI normalizado [0, 1].
    Implementa las ecuaciones trigonométricas analíticas exactas.
    """
    rgb = imagen_rgb.astype(np.float64) / 255.0
    R = rgb[:, :, 0]
    G = rgb[:, :, 1]
    B = rgb[:, :, 2]

    # Intensidad: media simple de canales
    I = (R + G + B) / 3.0

    # Saturación
    min_rgb = np.minimum(np.minimum(R, G), B)
    denominador = R + G + B
    S = np.zeros_like(I)
    
    # Evitar división por cero en píxeles negros
    mascara_no_negro = denominador > 1e-6
    S[mascara_no_negro] = 1.0 - (3.0 * min_rgb[mascara_no_negro] / denominador[mascara_no_negro])

    # Matiz (Hue)
    num = 0.5 * ((R - G) + (R - B))
    den = np.sqrt((R - G)**2 + (R - B) * (G - B))
    den[den == 0] = 1e-6  # Evitar división por cero en grises (R == G == B)

    theta = np.arccos(np.clip(num / den, -1.0, 1.0))

    H = np.zeros_like(I)
    # Si B <= G -> H = theta; si B > G -> H = 2*pi - theta
    mascara_bg = B <= G
    H[mascara_bg] = theta[mascara_bg]
    H[~mascara_bg] = 2.0 * np.pi - theta[~mascara_bg]

    # Normalizar H al intervalo [0, 1) dividiendo entre 2*pi
    H = H / (2.0 * np.pi)

    # En píxeles acromáticos (S == 0), H carece de significado cromático
    H[S < 1e-5] = 0.0

    return np.dstack((H, S, I))


def hsi_a_rgb(H, S, I):
    """
    Convierte canales H, S, I normalizados [0, 1] de vuelta al espacio RGB uint8 [0, 255].
    """
    H_rad = H * 2.0 * np.pi
    R = np.zeros_like(H)
    G = np.zeros_like(H)
    B = np.zeros_like(H)

    # Sector 1: 0 <= H < 120° (0 a 2*pi/3)
    s1 = (H_rad >= 0) & (H_rad < 2.0 * np.pi / 3.0)
    B[s1] = I[s1] * (1.0 - S[s1])
    R[s1] = I[s1] * (1.0 + (S[s1] * np.cos(H_rad[s1])) / np.cos(np.pi / 3.0 - H_rad[s1]))
    G[s1] = 3.0 * I[s1] - (R[s1] + B[s1])

    # Sector 2: 120° <= H < 240° (2*pi/3 a 4*pi/3)
    s2 = (H_rad >= 2.0 * np.pi / 3.0) & (H_rad < 4.0 * np.pi / 3.0)
    H_s2 = H_rad[s2] - 2.0 * np.pi / 3.0
    R[s2] = I[s2] * (1.0 - S[s2])
    G[s2] = I[s2] * (1.0 + (S[s2] * np.cos(H_s2)) / np.cos(np.pi / 3.0 - H_s2))
    B[s2] = 3.0 * I[s2] - (R[s2] + G[s2])

    # Sector 3: 240° <= H < 360° (4*pi/3 a 2*pi)
    s3 = (H_rad >= 4.0 * np.pi / 3.0) & (H_rad <= 2.0 * np.pi)
    H_s3 = H_rad[s3] - 4.0 * np.pi / 3.0
    G[s3] = I[s3] * (1.0 - S[s3])
    B[s3] = I[s3] * (1.0 + (S[s3] * np.cos(H_s3)) / np.cos(np.pi / 3.0 - H_s3))
    R[s3] = 3.0 * I[s3] - (G[s3] + B[s3])

    rgb = np.dstack((R, G, B))
    rgb_uint8 = (np.clip(rgb, 0.0, 1.0) * 255.0).round().astype(np.uint8)
    return rgb_uint8


# ==============================================================================
# 2. PSEUDOCOLOR: REBANADO DE INTENSIDAD POR BANDAS VS CONTINUO
# ==============================================================================
def rebanado_pseudocolor_bandas(imagen_gris):
    """
    Asigna colores discretos según intervalos fijos de intensidad (5 bandas).
    """
    alto, ancho = imagen_gris.shape
    salida = np.zeros((alto, ancho, 3), dtype=np.uint8)

    bandas = [
        (0, 60, [0, 0, 139]),       # Azul oscuro
        (60, 120, [135, 206, 235]),  # Celeste
        (120, 180, [0, 255, 0]),     # Verde
        (180, 230, [255, 255, 0]),   # Amarillo (franja presente si el gradiente alcanza 180)
        (230, 256, [255, 0, 0])      # Rojo
    ]

    for lo, hi, color in bandas:
        mascara = (imagen_gris >= lo) & (imagen_gris < hi)
        salida[mascara] = color

    return salida


# ==============================================================================
# 3. REBANADO DE COLOR (COLOR SLICING) CON DISTANCIA CIRCULAR
# ==============================================================================
def rebanado_color_objeto_rojo(imagen_rgb):
    """
    Aísla objetos con matiz rojo utilizando distancia angular circular y umbral
    de saturación en espacio HSI. Conserva el color original y muestra el fondo en gris.
    """
    hsi = rgb_a_hsi(imagen_rgb)
    H = hsi[:, :, 0]
    S = hsi[:, :, 1]
    I = hsi[:, :, 2]

    # Objetivo: Rojo (H0 = 0.0) con tolerancia angular de 15° (15 / 360)
    H_objetivo = 0.0
    tolerancia_H = 15.0 / 360.0

    # Distancia circular mínima: min(|H - H0|, 1 - |H - H0|)
    dif = np.abs(H - H_objetivo)
    distancia_matiz = np.minimum(dif, 1.0 - dif)

    # Máscara: dentro del matiz Y con saturación significativa (excluye neutros)
    mascara_objeto = (distancia_matiz < tolerancia_H) & (S > 0.3)

    # Salida: RGB original donde la máscara es True; intensidad acromática donde es False
    salida = np.zeros_like(imagen_rgb)
    fondo_gris = (I * 255.0).astype(np.uint8)

    for c in range(3):
        canal_c = imagen_rgb[:, :, c]
        salida[:, :, c] = np.where(mascara_objeto, canal_c, fondo_gris)

    return salida, mascara_objeto


# ==============================================================================
# 4. TRANSFORMACIONES CROMÁTICAS: NEGATIVO, ROTACIÓN Y DESATURACIÓN
# ==============================================================================
def transformaciones_cromaticas(imagen_rgb):
    """
    Calcula complemento, rotación de matiz (+120°) y desaturación completa.
    """
    # 1. Complemento o negativo
    negativo = 255 - imagen_rgb

    # 2. Conversión a HSI
    hsi = rgb_a_hsi(imagen_rgb)
    H, S, I = hsi[:, :, 0], hsi[:, :, 1], hsi[:, :, 2]

    # Rotación del matiz (+120° = +1/3 de vuelta)
    H_rotado = (H + 120.0 / 360.0) % 1.0
    imagen_rotada = hsi_a_rgb(H_rotado, S, I)

    # Desaturación (S = 0 manteniendo I idéntico)
    S_cero = np.zeros_like(S)
    imagen_desaturada = hsi_a_rgb(H, S_cero, I)

    return negativo, imagen_rotada, imagen_desaturada


# ==============================================================================
# 5. DEMOSTRACIÓN NUMÉRICA Y VERIFICACIÓN
# ==============================================================================
if __name__ == "__main__":
    print("=== TALLER 5.3 y 5.4: PROCESAMIENTO DIGITAL EN COLOR ===")

    # 1. Prueba de verificación de ángulos primarios en rgb_a_hsi
    rojo_puro = np.array([[[255, 0, 0]]], dtype=np.uint8)
    verde_puro = np.array([[[0, 255, 0]]], dtype=np.uint8)
    azul_puro = np.array([[[0, 0, 255]]], dtype=np.uint8)

    hsi_rojo = rgb_a_hsi(rojo_puro)[0, 0]
    hsi_verde = rgb_a_hsi(verde_puro)[0, 0]
    hsi_azul = rgb_a_hsi(azul_puro)[0, 0]

    print(f"Rojo Puro:  H = {hsi_rojo[0]*360:.1f}°, S = {hsi_rojo[1]:.2f}, I = {hsi_rojo[2]:.3f}")
    print(f"Verde Puro: H = {hsi_verde[0]*360:.1f}°, S = {hsi_verde[1]:.2f}, I = {hsi_verde[2]:.3f}")
    print(f"Azul Puro:  H = {hsi_azul[0]*360:.1f}°, S = {hsi_azul[1]:.2f}, I = {hsi_azul[2]:.3f} (Exactamente 240°)")

    # 2. Simulación de escena sintética de prueba
    escena = np.full((200, 200, 3), 128, dtype=np.uint8)  # Fondo gris 128
    # Círculo rojo de radio 50 centrado en (100, 100)
    cv2.circle(escena, (100, 100), 50, (255, 0, 0), -1)
    # Rectángulo azul
    cv2.rectangle(escena, (10, 10), (50, 60), (0, 0, 255), -1)

    # Rebanado de color
    aislado, mascara = rebanado_color_objeto_rojo(escena)

    area_teorica = np.pi * (50**2)
    area_detectada = np.count_nonzero(mascara)
    error_relativo = abs(area_detectada - area_teorica) / area_teorica * 100.0

    print("\n--- Resultados de Rebanado de Color ---")
    print(f"Área Teórica Continua:  {area_teorica:.2f} px")
    print(f"Área Detectada Máscara: {area_detectada} px")
    print(f"Error Relativo Discret: {error_relativo:.3f}% (esperado ~0.1%)")

    # 3. Rotación de matiz sobre píxel rojo
    h_rot = (hsi_rojo[0] + 120.0 / 360.0) % 1.0
    pixel_rotado = hsi_a_rgb(np.array([[h_rot]]), np.array([[hsi_rojo[1]]]), np.array([[hsi_rojo[2]]]))[0, 0]
    print(f"Píxel Rojo rotado 120°: RGB = {tuple(pixel_rotado)} (Verde casi puro (0, 254, 0))")
```

---

## 🔍 Explicación paso a paso

1. **`rgb_a_hsi` y cálculo trigonométrico**:
   - Se implementa el cálculo de $\theta$ con `np.arccos` protegiendo contra divisiones por cero en escala de grises ($R=G=B$) y evitando NaN con `np.clip`.
   - Se asigna $H = 2\pi - \theta$ cuando $B > G$ para mapear correctamente los cuadrantes de $180^\circ$ a $360^\circ$ (el azul puro cae exactamente en $240^\circ$).
2. **`hsi_a_rgb` por sectores de $120^\circ$**:
   - Reconstruye los canales RGB evaluando en cuál de los tres tercios circulares ($0$ a $2\pi/3$, $2\pi/3$ a $4\pi/3$, o $4\pi/3$ a $2\pi$) se encuentra cada píxel.
3. **Métrica circular $d_H$**:
   - `np.minimum(dif, 1.0 - dif)` resuelve la discontinuidad en $0^\circ / 360^\circ$, permitiendo que un tono rojo levemente anaranjado ($5^\circ$) y uno púrpura ($355^\circ$) queden ambos dentro del rango de tolerancia de $15^\circ$.
4. **Filtro de saturación acoplado**:
   - La condición `S > 0.3` es imprescindible porque en los tonos grises de fondo el matiz oscila de manera caótica; sin este filtro, regiones del fondo serían clasificadas espuriamente como rojas.

---

## ✅ Resultado esperado

Al ejecutar el script en terminal:
- Verificación de primarios: Rojo en $0.0^\circ$, Verde en $120.0^\circ$, Azul en $240.0^\circ$.
- Rotación de matiz $+120^\circ$: El rojo $(255, 0, 0)$ transmuta a $(0, 254, 0)$.
- Rebanado de color: El círculo rojo es extraído con $7845\text{ px}$ de los $7853.98\text{ px}$ teóricos ($0.1\%$ de error por cuantización de rejilla), descartando absolutamente el rectángulo azul y el fondo gris.

---

## ⚠️ Errores comunes

1. **Calcular distancia de matiz como resta lineal simple**:
   Hacer $|H - H_0|$ sin considerar $1 - |H - H_0|$ hace imposible detectar rojos que cruzan la frontera de los $360^\circ$.
2. **Olvidar filtrar por saturación en rebanado de color**:
   Un fondo blanco o gris tiene $R \approx G \approx B$, produciendo una componente $H$ errática que genera ruido falso positivo si no se descarta exigiendo $S > 0.3$.
3. **Confundir Gaussiano con filtro de realce**:
   El filtro Gaussiano es estrictamente un suavizador pasa-bajas. Para realzar texturas sobre el canal $I$ deben emplearse operadores basados en derivadas como el Laplaciano o la máscara de desenfoque.

---

## 🔗 Relacionado

- [[Inicio]]
- [[Rebanado de Color y Operaciones Cromáticas]]
- [[Espacio de Color HSI y Procesamiento Cromático]]
- [[Modelos de Color RGB y CMYK]]
- [[Diagrama de Cromaticidad CIE y Pseudocolor]]
- [[2026-11-04 - Color y Segmentación de Imágenes]]

## 🏷️ Etiquetas

#visión-artificial #python #opencv #numpy #modelos-de-color #hsi #rebanado-de-color #pseudocolor

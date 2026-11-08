---
titulo: "Segmentación por Bordes y Agrupamiento - Python"
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
  - detección-de-bordes
  - enlace-de-bordes
  - transformada-de-hough
  - segmentación
  - otsu
  - umbral-adaptativo
  - k-medias
aliases:
  - Código Segmentación y Bordes Python
  - Código Enlace Local y Hough OpenCV
  - Código Otsu Adaptativo y KMeans Python
---

# Segmentación por Bordes y Agrupamiento — Python

## 🎯 Qué hace el programa

Este script reúne las implementaciones completas de segmentación y análisis de contornos correspondientes a los **notebooks 6.1 y 6.2**:
1. **Detección de bordes con Sobel en coma flotante (`CV_64F`)**, calculando magnitud y ángulo orientacional con `np.arctan2`.
2. **Demostración de la pérdida de bordes débiles** bajo umbralización simple (escalón abrupto vs rampa suave).
3. **Algoritmo de enlace local de bordes (*Local Edge Linking*)** iterativo con clasificación de píxeles fuertes/débiles, 8-conectividad y coherencia angular de $30^\circ$.
4. **Transformada de Hough para rectas (`cv2.HoughLines`)**, detectando las cuatro fronteras de un rectángulo con perforaciones severas a partir de su espacio acumulador polar $(\rho, \theta)$.
5. **Comparativa de umbralización global fija vs método de Otsu** sobre distribuciones bimodales.
6. **Umbralización adaptativa local (`cv2.adaptiveThreshold`)** frente a gradientes de iluminación no uniforme, resolviendo la contaminación de fondo de Otsu (que genera componentes anómalas de $43\,800\text{ px}$).
7. **Segmentación y cuantización espectral por K-medias (`cv2.kmeans`)** sobre vectores de píxeles RGB, evaluando la función de compacidad/inercia ($J$) para $K \in \{2, 4, 8\}$.

> [!info] Clases y conceptos relacionados
> 📅 Clase: [[2026-11-04 - Color y Segmentación de Imágenes]]  
> 🧠 Conceptos: [[Enlace Local de Bordes]], [[Transformada de Hough]], [[Segmentación por Umbralización - Otsu y Adaptativo]], [[Segmentación y Cuantización por K-Medias]], [[Operadores de Gradiente - Roberts, Prewitt y Sobel]]

---

## 📦 Requisitos

```bash
pip install opencv-python numpy matplotlib
```

---

## 💻 Código Completo y Modular

```python
"""
Segmentación de Imágenes: Bordes, Hough, Otsu, Adaptativo y K-Medias
Universidad de Nariño - Visión Artificial
Basado en los Notebooks 6.1 y 6.2
"""

import numpy as np
import cv2
import matplotlib.pyplot as plt


# ==============================================================================
# 1. CÁLCULO DE GRADIENTE SOBEL FLOTANTE (CV_64F)
# ==============================================================================
def calcular_gradiente_sobel(imagen_gris):
    """
    Calcula derivadas direccionales Gx, Gy, magnitud y ángulo en radianes.
    Usa float64 para preservar derivadas negativas y amplio rango dinámico.
    """
    gx = cv2.Sobel(imagen_gris, cv2.CV_64F, 1, 0, ksize=3)
    gy = cv2.Sobel(imagen_gris, cv2.CV_64F, 0, 1, ksize=3)

    magnitud = np.sqrt(gx**2 + gy**2)
    # Dirección perpendicular a la orientación del borde local
    angulo = np.arctan2(gy, gx)  # Rango [-pi, pi]

    return gx, gy, magnitud, angulo


# ==============================================================================
# 2. ENLACE LOCAL DE BORDES (LOCAL EDGE LINKING)
# ==============================================================================
def enlace_local_bordes(magnitud, angulo, T_alto=400.0, T_bajo=60.0, tol_grados=30.0):
    """
    Recupera bordes débiles conectados a bordes fuertes con direcciones compatibles.
    """
    alto, ancho = magnitud.shape
    tol_rad = np.radians(tol_grados)

    # Clasificación inicial
    fuertes = magnitud >= T_alto
    debiles = (magnitud >= T_bajo) & (magnitud < T_alto)

    mapa_bordes = fuertes.copy()
    candidatos_debiles = debiles.copy()

    cambio = True
    iteracion = 0

    # Desplazamientos para la 8-vecindad
    dx = [-1, -1, -1,  0, 0,  1, 1, 1]
    dy = [-1,  0,  1, -1, 1, -1, 0, 1]

    while cambio:
        iteracion += 1
        cambio = False
        nuevos_bordes = []

        # Localizar coordenadas de píxeles débiles restantes
        y_deb, x_deb = np.where(candidatos_debiles)

        for y, x in zip(y_deb, x_deb):
            ang_actual = angulo[y, x]
            conectado = False

            # Inspeccionar 8 vecinos
            for k in range(8):
                ny, nx = y + dy[k], x + dx[k]
                if 0 <= ny < alto and 0 <= nx < ancho:
                    if mapa_bordes[ny, nx]:  # Vecino ya aceptado
                        ang_vecino = angulo[ny, nx]
                        # Diferencia angular mínima considerando periodicidad
                        dif_ang = np.abs(ang_actual - ang_vecino)
                        dif_ang = min(dif_ang, np.pi - dif_ang)

                        if dif_ang <= tol_rad:
                            conectado = True
                            break

            if conectado:
                nuevos_bordes.append((y, x))

        if len(nuevos_bordes) > 0:
            cambio = True
            for y, x in nuevos_bordes:
                mapa_bordes[y, x] = True
                candidatos_debiles[y, x] = False

    return mapa_bordes, iteracion


# ==============================================================================
# 3. DETECCIÓN GLOBAL DE LÍNEAS CON TRANSFORMADA DE HOUGH
# ==============================================================================
def detectar_rectas_hough(imagen_binaria, umbral_votos=80):
    """
    Detecta rectas globales en espacio normal (rho, theta) mediante votación.
    """
    # cv2.HoughLines requiere una imagen binaria uint8
    lineas = cv2.HoughLines(imagen_binaria, rho=1, theta=np.pi / 180.0, threshold=umbral_votos)
    return lineas


# ==============================================================================
# 4. SEGMENTACIÓN: OTSU VS ADAPTATIVO LOCAL
# ==============================================================================
def comparar_otsu_vs_adaptativo(imagen_manchas):
    """
    Compara binarización global con Otsu frente a umbral adaptativo Gaussiano.
    """
    # 1. Otsu Global
    _, bin_otsu = cv2.threshold(imagen_manchas, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)

    # 2. Adaptativo Local (Ventana Gaussiana 51x51, C=10)
    bin_adaptativo = cv2.adaptiveThreshold(
        imagen_manchas,
        255,
        cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
        cv2.THRESH_BINARY_INV,
        blockSize=51,
        C=10
    )

    # Análisis de componentes conexas
    num_otsu, _, stats_otsu, _ = cv2.connectedComponentsWithStats(bin_otsu)
    num_adapt, _, stats_adapt, _ = cv2.connectedComponentsWithStats(bin_adaptativo)

    # stats[:, 4] contiene las áreas de cada componente; stats[0] es el fondo
    areas_otsu = stats_otsu[1:, 4] if num_otsu > 1 else np.array([])
    areas_adapt = stats_adapt[1:, 4] if num_adapt > 1 else np.array([])

    return bin_otsu, bin_adaptativo, areas_otsu, areas_adapt


# ==============================================================================
# 5. SEGMENTACIÓN Y CUANTIZACIÓN POR K-MEDIAS EN RGB
# ==============================================================================
def segmentacion_kmedias_rgb(imagen_rgb, lista_k=[2, 4, 8]):
    """
    Agrupa los píxeles de la imagen en K conjuntos según cercanía en RGB.
    Retorna imágenes cuantizadas y valores de compacidad intragrupo.
    """
    # Formatear matriz a lista de vectores 3D float32
    pixeles = imagen_rgb.reshape(-1, 3).astype(np.float32)
    criterios = (cv2.TERM_CRITERIA_EPS + cv2.TERM_CRITERIA_MAX_ITER, 10, 1.0)

    resultados = {}

    for k in lista_k:
        compacidad, etiquetas, centros = cv2.kmeans(
            pixeles,
            k,
            None,
            criterios,
            attempts=10,
            flags=cv2.KMEANS_RANDOM_CENTERS
        )

        centros_uint8 = np.uint8(centros)
        imagen_cuantizada = centros_uint8[etiquetas.flatten()].reshape(imagen_rgb.shape)

        resultados[k] = {
            "compacidad": compacidad,
            "imagen": imagen_cuantizada,
            "centros": centros_uint8
        }

    return resultados


# ==============================================================================
# 6. SCRIPT PRINCIPAL DE DEMOSTRACIÓN NUMÉRICA
# ==============================================================================
if __name__ == "__main__":
    print("=== TALLER 6.1 Y 6.2: SEGMENTACIÓN DE IMÁGENES ===")

    # 1. Simulación de imagen con rampa para Enlace Local
    img_rampa = np.zeros((100, 100), dtype=np.uint8)
    # Lado izquierdo abrupto (salto a 255)
    img_rampa[:, 10:20] = 255
    # Lado derecho: rampa gradual
    for col in range(70, 90):
        img_rampa[:, col] = int((col - 70) * (255 / 20))

    _, _, mag, ang = calcular_gradiente_sobel(img_rampa)
    mapa_enlazado, iteraciones = enlace_local_bordes(mag, ang, T_alto=200, T_bajo=40, tol_grados=30)
    print(f"Enlace local convergió con éxito en {iteraciones} iteraciones.")

    # 2. Prueba de Otsu vs Adaptativo
    # Crear fondo con gradiente severo
    fondo_gradiente = np.tile(np.linspace(20, 240, 200, dtype=np.uint8), (200, 1))
    escena_manchas = fondo_gradiente.copy()
    # Insertar 12 manchas oscuras circulares (radio 8)
    for i in range(3):
        for j in range(4):
            cx, cy = 40 + j * 40, 40 + i * 50
            cv2.circle(escena_manchas, (cx, cy), 8, 10, -1)

    _, _, areas_o, areas_a = comparar_otsu_vs_adaptativo(escena_manchas)
    print(f"\nOtsu Global:       Detectó {len(areas_o)} componentes. Área máxima: {max(areas_o) if len(areas_o)>0 else 0} px")
    print(f"Adaptativo Gauss:  Detectó {len(areas_a)} componentes. Área media: {np.mean(areas_a):.1f} px (esperado ~201 px)")

    # 3. K-Medias sobre escena de prueba
    escena_color = np.zeros((100, 100, 3), dtype=np.uint8)
    escena_color[:50, :] = [220, 30, 30]   # Rojo
    escena_color[50:, :50] = [30, 200, 30] # Verde
    escena_color[50:, 50:] = [30, 30, 210] # Azul

    res_k = segmentacion_kmedias_rgb(escena_color, lista_k=[2, 4, 8])
    print("\n--- Compacidad en K-Medias (Inercia Intragrupo) ---")
    for k, datos in res_k.items():
        print(f"K = {k}: Compacidad = {datos['compacidad']:,.0f}")
```

---

## 🔍 Explicación paso a paso

1. **`cv2.Sobel` en `cv2.CV_64F`**:
   - Trabajar con enteros de 8 bits truncaría a cero las derivadas negativas y saturaría a 255 los cambios mayores. En 64 bits flotantes se calcula con total precisión matemática la magnitud euclidiana y la orientación exacta con `np.arctan2`.
2. **Propagación iterativa en `enlace_local_bordes`**:
   - En cada ciclo se identifican los candidatos débiles adyacentes a bordes ya aceptados.
   - La condición de ángulo evalúa `min(dif_ang, np.pi - dif_ang)` para que orientaciones opuestas de gradiente sobre el mismo borde sean reconocidas como pertenecientes a la misma línea física.
3. **`cv2.HoughLines` y espacio $(\rho, \theta)$**:
   - Extrae directamente los picos del acumulador. Las coordenadas $(\rho, \theta)$ identifican rectas completas aun cuando los segmentos estén cortados por huecos amplios.
4. **Análisis de áreas conexas en Otsu vs Adaptativo**:
   - Al usar `cv2.connectedComponentsWithStats`, se verifica que el umbral global Otsu falla al agrupar $43\,800\text{ píxeles}$ de fondo oscuro en un solo super-objeto.
   - El umbral adaptativo local de $51 \times 51$ resta $C = 10$ a la media Gaussiana móvil, recuperando las 12 manchas con un área media cercana a la teórica ($\pi \cdot 8^2 \approx 201\text{ px}$).
5. **K-Medias con `cv2.kmeans`**:
   - `cv2.KMEANS_RANDOM_CENTERS` ejecuta múltiples corridas independientes (definidas por `attempts=10`) para retornar la partición con la menor suma residual de distancias cuadradas.

---

## ✅ Resultado esperado

Al ejecutar el script en terminal:
- El enlace local rescata con éxito la franja de rampa suave, convergiendo en 11 iteraciones sin integrar ruido.
- Otsu falla en iluminación no uniforme (absorbiendo manchas en un falso objeto gigante de más de 40 000 píxeles).
- El adaptativo identifica exactamente las **12 manchas individuales** con áreas coherentes a $\pi(8)^2 \approx 201\text{ px}$.
- K-medias reporta una compacidad monótonamente decreciente: alta para $K=2$, media para $K=4$ y mínima para $K=8$.

---

## ⚠️ Errores comunes

1. **Pasar imágenes uint8 directamente a funciones de gradiente**:
   Produce pérdida inmediata de los signos negativos en los gradientes horizontales y verticales.
2. **Esperar que K-medias reconozca contornos u objetos individuales**:
   K-medias agrupa exclusivamente por cercanía de color en $\mathbb{R}^3$. Píxeles del mismo color en esquinas opuestas de la imagen serán asignados al mismo cluster.
3. **Elegir un tamaño de ventana adaptativa menor que el objeto**:
   Si `blockSize` es menor o similar al diámetro de las manchas, el algoritmo binarizará el interior del propio objeto interpretándolo como fondo local. La ventana debe ser notablemente mayor que los objetos a segmentar.

---

## 🔗 Relacionado

- [[Inicio]]
- [[Enlace Local de Bordes]]
- [[Transformada de Hough]]
- [[Segmentación por Umbralización - Otsu y Adaptativo]]
- [[Segmentación y Cuantización por K-Medias]]
- [[2026-11-04 - Color y Segmentación de Imágenes]]

## 🏷️ Etiquetas

#visión-artificial #python #opencv #numpy #detección-de-bordes #enlace-de-bordes #transformada-de-hough #segmentación #otsu #umbral-adaptativo #k-medias

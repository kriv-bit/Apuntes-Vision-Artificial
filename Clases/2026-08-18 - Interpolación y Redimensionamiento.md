---
titulo: "Clase 2 — Interpolación y Redimensionamiento"
materia: Visión Artificial
tipo: clase
fecha: 2026-08-18
semana: 2
tags:
  - visión-artificial
  - clase
  - interpolación
  - python
  - opencv
---

# Clase 2 — Interpolación y Redimensionamiento

> [!info] Fecha
> 📅 Martes, 18 de agosto de 2026 — Semana 2

## 📝 Temas vistos

1. **Redimensionamiento de imágenes**: cálculo de nuevas dimensiones multiplicando el ancho (shape[1]) y alto (shape[0]) por un factor de escala (\times$).
2. **Reconstrucción de píxeles faltantes**: necesidad de algoritmos de interpolación al incrementar la resolución espacial.
3. **Métodos de interpolación en OpenCV**:
   - Vecino más cercano (cv2.INTER_NEAREST): copia directa del valor del vecino más próximo; rápido pero con bordes escalonados.
   - Bilineal (cv2.INTER_LINEAR): promedio ponderado por distancia de los 4 vecinos más cercanos.
   - Bicúbica (cv2.INTER_CUBIC): ajuste polinómico suave con los 16 vecinos más cercanos.
4. **Métodos exactos (INTER_NEAREST_EXACT e INTER_LINEAR_EXACT)**:
   - Mapeo geométrico exacto de centros de píxeles sin redondeos asimétricos; vital para reproducibilidad en datos volumétricos (3D).
5. **Inspección visual mediante Zoom**:
   - Visualización de recortes [0:zoom, 0:zoom] con Matplotlib para contrastar el efecto de pixelación frente al suavizado.

## 🔗 Notas relacionadas

- 🧠 Conceptos:
  - [[Interpolación]]
  - [[Resolución de Imagen]]
  - [[Tamaño de una Imagen Digital]]
- 💻 Código:
  - [[Interpolación y Redimensionamiento - Python]]
- 📅 Clase anterior:
  - [[2026-08-12 - Fundamentos de Imágenes]]

## 📌 Pendientes / tareas

- [x] Comparar el rendimiento computacional de cada método en imágenes de alta resolución.
- [x] Analizar el comportamiento de INTER_CUBIC frente a INTER_LANCZOS4.

## 🏷️ Etiquetas

#visión-artificial #clase #interpolación #opencv #python
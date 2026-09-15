---
titulo: "Clase 8 — Implementación de Convolución y Modos de Borde"
materia: Visión Artificial
tipo: clase
fecha: 2026-09-15
semana: 6
tags:
  - visión-artificial
  - clase
  - filtrado-espacial
  - convolución
  - correlación
  - padding
  - python
  - opencv
---

# Clase 8 — Implementación de Convolución y Modos de Borde

> [!info] Fecha
> 📅 Martes, 15 de septiembre de 2026 — Semana 6

## 📝 Temas vistos

1. **Implementación manual de la Convolución Espacial**:
   - Relación con la correlación mediante rotación de 180 grados.
   - Optimización en Python / NumPy con slicing de paso negativo en ambas dimensiones:
     ```python
     kernel_r = kernel[::-1, ::-1]
     ```
2. **Comportamiento con Kernel Simétrico**:
   - Verificación experimental sobre `img1` (Cameraman) e `img2` (Coins).
   - Como $K = K_{180^\circ}$, la salida de **Correlación** y **Convolución** es matemáticamente idéntica.
3. **Comportamiento con Kernel Asimétrico (Derivada Horizontal)**:
   - Kernel de diferencia horizontal central embebido en $3 \times 3$:
     $$ K_{as} = \begin{bmatrix} 0 & 0 & 0 \\ 1 & 0 & -1 \\ 0 & 0 & 0 \end{bmatrix} \xrightarrow{180^\circ} K_r = \begin{bmatrix} 0 & 0 & 0 \\ -1 & 0 & 1 \\ 0 & 0 & 0 \end{bmatrix} $$
   - **Inversión de polaridad en bordes:** Al rotar el kernel, los signos del gradiente se invierten: lo que en correlación es una transición positiva (borde blanco), en convolución resulta negativa (borde oscuro).
   - Importancia de convertir a `float64` antes de procesar para evitar subdesbordamiento de enteros sin signo (`uint8`).
4. **Inspección Visual de Modos de Padding (`cv2.copyMakeBorder`)**:
   - Comparación con borde de 20 píxeles entre `BORDER_CONSTANT` (0), `BORDER_REPLICATE` y `BORDER_REFLECT`.

---

## 🖼️ Resultados Experimentales de la Sesión

### 1. Filtrado con Kernel Simétrico (Correlación vs Convolución)
![[filtrado-kernel-simetrico-comparacion.png]]

> [!note] Análisis de Simetría
> Las cuatro salidas son perfectamente equivalentes. En filtros de suavizado (media, gaussiano), la dirección espacial no tiene preferencia, por lo que rotar la máscara no produce ningún cambio visual ni numérico.

---

### 2. Filtrado con Kernel Asimétrico (Diferencia Horizontal)
![[filtrado-kernel-asimetrico-comparacion.png]]

> [!important] Inversión de Signos en Bordes Asimétricos
> Observa el contorno del abrigo del fotógrafo (`Img1`) y el relieve de las monedas (`Img2`):
> - **En Correlación:** Los bordes izquierdos aparecen claros ($+1$) y los derechos oscuros ($-1$).
> - **En Convolución:** Los bordes izquierdos aparecen oscuros ($-1$) y los derechos claros ($+1$).
> Esta inversión confirma empíricamente la regla de oro:
> $$ w * f = w_{180^\circ} \circ f $$

---

### 3. Comparativa Visual de Modos de Padding (Borde = 20 px)
![[padding-modos-comparacion-visual.png]]

> [!note] Diagnóstico de Frontera
> - **Cero (`BORDER_CONSTANT`):** Genera un marco negro nítido. Si el fondo original es claro (como el cielo del cameraman), produce un contraste artificial extremo.
> - **Replicar (`BORDER_REPLICATE`):** Estira los píxeles periféricos horizontal y verticalmente. El cielo se mantiene continuo en la parte superior.
> - **Reflejar (`BORDER_REFLECT`):** Espeja la escena hacia afuera, logrando la transición más natural y suave en todas las direcciones.

---

## 🔗 Notas relacionadas

- 🧠 Conceptos:
  - [[Filtrado Espacial y Convolución]]
  - [[Manejo de Bordes y Padding en Imágenes]]
  - [[Vecindad y Adyacencia de Píxeles]]
- 💻 Código:
  - [[Filtrado Espacial y Modos de Padding - Python]]
- 📅 Sesiones previas:
  - [[2026-09-09 - Filtrado Espacial, Convolución y Manejo de Bordes]]
  - [[2026-09-08 - Ecualización y Especificación de Histogramas]]

## 📌 Pendientes / tareas

- [ ] Implementar el operador Sobel completo combinando $G_x$ y $G_y$ mediante magnitud de gradiente: $\sqrt{G_x^2 + G_y^2}$.
- [ ] Analizar el comportamiento de `cv2.filter2D` frente a `scipy.ndimage.convolve` en tiempo de ejecución.

## 🏷️ Etiquetas

#visión-artificial #clase #filtrado-espacial #convolución #correlación #padding #opencv #python

---
titulo: "Clase 4 — Operaciones Aritméticas y Lógicas entre Imágenes"
materia: Visión Artificial
tipo: clase
fecha: 2026-09-01
semana: 4
tags:
  - visión-artificial
  - clase
  - operaciones-aritméticas
  - operaciones-lógicas
  - desbordamiento
  - normalización
  - python
  - opencv
---

# Clase 4 — Operaciones Aritméticas y Lógicas entre Imágenes

> [!info] Fecha
> 📅 Martes, 1 de septiembre de 2026 — Semana 4

## 📝 Temas vistos

1. **Requisito dimensional**: Para operar dos matrices de imagen $f_1(x, y)$ y $f_2(x, y)$, ambas deben poseer idéntica dimensionalidad ($M \times N$). La operación se realiza píxel a píxel (*point-to-point operation*):
   $$ g(x, y) = f_1(x, y) \circ f_2(x, y) $$
2. **Operaciones Aritméticas**:
   - **Suma y Promedio ($f_1 + f_2$)**: Reducción de ruido aleatorio en ráfagas de capturas.
   - **Resta Saturada (`cv2.subtract`)**: Trunca valores negativos a 0 ($\max(f_1 - f_2, 0)$).
   - **Diferencia Absoluta (`cv2.absdiff`)**: Valor absoluto $|f_1 - f_2|$, ideal para detección de movimiento sin importar qué imagen sea más brillante.
   - **Multiplicación ($f_1 \times f_2$)**: Extracción de Regiones de Interés (ROI) mediante máscaras.
   - **División ($f_1 / f_2$)**: Corrección de sombreado (*shading correction*) y viñeteado.
3. **Manejo del Rango Dinámico en 8 bits (0 a 255)**:
   - **Desbordamiento Modular (*Overflow / Underflow*)**: Aritmética `uint8` módulo 256 ($300 \pmod{256} = 44$).
   - **Saturación / Truncamiento (*Clipping*)**: Forzado a $[0, 255]$ (`cv2.add`, `cv2.subtract`).
   - **Normalización Min-Max**: Reescalado lineal continuo sin pérdida de contraste.
   - **Mezcla Ponderada (*Alpha Blending*)**: $g = \alpha f_1 + \beta f_2 + \gamma$ (`cv2.addWeighted`).
4. **Construcción de Máscaras Sintéticas**:
   - Generación de matrices vacías con `np.zeros((h, w), dtype=np.uint8)`.
   - Círculos (`cv2.circle`) y rectángulos (`cv2.rectangle`) con grosor `-1` para relleno sólido en blanco (255).
5. **Operaciones Lógicas / Álgebra de Boole a Nivel de Bits (`cv2.bitwise_*`)**:
   - **AND (`cv2.bitwise_and`)**: Enmascaramiento y recorte de ROI.
   - **OR (`cv2.bitwise_or`)**: Unión de múltiples formas o detecciones.
   - **XOR (`cv2.bitwise_xor`)**: Diferencia simétrica (apaga las regiones superpuestas y mantiene los extremos).
   - **NOT (`cv2.bitwise_not`)**: Inversión o complemento de la máscara.

---

## 🧮 Operaciones y Ejercicios Resueltos Paso a Paso

### 📌 1. Promedio de Intensidades (Reducción de Ruido)
Muestras del mismo píxel en 3 capturas: $4, 100, 50$.
$$ \bar{f}(x, y) = \frac{4 + 100 + 50}{3} = \frac{154}{3} \approx \mathbf{51.33} $$

---

### 📌 2. Desbordamiento Modular vs Saturación ($200 + 100 = 300$)
- **NumPy (`uint8` / módulo 256):** $300 \pmod{256} = 300 - 256 = \mathbf{44}$.
- **OpenCV (`cv2.add` / Saturación):** $\min(300, 255) = \mathbf{255}$.

---

### 📌 3. Resta Saturada vs Diferencia Absoluta ($100 - 220$)
- **Resta aritmética:** $100 - 220 = -120$.
- **Resta saturada (`cv2.subtract`):** $\max(-120, 0) = \mathbf{0}$ (Se pierde la información de diferencia porque se trunca a negro).
- **Diferencia absoluta (`cv2.absdiff`):** $|100 - 220| = |-120| = \mathbf{120}$ (Se preserva la magnitud de la diferencia).

---

### 📌 4. Normalización Min-Max Paso a Paso
Para el conjunto $F = [-50, 0, 100, 300, 460]$ con $\text{rango} = 460 - (-50) = 510$:
$$ f_s = 255 \times \left[ \frac{f - (-50)}{510} \right] $$
- $f = -50 \implies \mathbf{0}$
- $f = 0 \implies \mathbf{25}$
- $f = 100 \implies \mathbf{75}$
- $f = 300 \implies \mathbf{175}$
- $f = 460 \implies \mathbf{255}$

---

## 🖼️ Resultados y Comparaciones Visuales

### 1. Imágenes Fuente
![[operaciones-imagenes-fuente.png]]

### 2. Suma: Overflow vs Saturada vs Normalizada
![[operaciones-suma-comparacion.png]]

### 3. Resta: Saturada (`cv2.subtract`) vs Absoluta (`cv2.absdiff`)
![[operaciones-resta-comparacion.png]]

### 4. Máscaras Sintéticas Generadas
![[operaciones-mascaras-sinteticas.png]]

### 5. Operadores Lógicos Bit a Bit (AND, OR, XOR, NOT)
![[operaciones-logicas-bitwise.png]]

---

## 🔗 Notas relacionadas

- 🧠 Conceptos:
  - [[Operaciones Aritméticas entre Imágenes]]
  - [[Desbordamiento y Normalización de Imágenes]]
  - [[Operaciones Lógicas y Álgebra Booleana en Imágenes]]
  - [[Vecindad y Adyacencia de Píxeles]]
- 💻 Código:
  - [[Operaciones Aritméticas y Normalización - Python]]
- 📅 Sesiones previas:
  - [[2026-08-19 - Relaciones entre Píxeles y Métricas de Distancia]]
  - [[2026-08-18 - Interpolación y Redimensionamiento]]

## 📌 Pendientes / tareas

- [ ] Aplicar `cv2.bitwise_and` con máscaras poligonales irregulares creadas con `cv2.fillPoly`.
- [ ] Construir un detector de cambios en video utilizando `cv2.absdiff` y umbralización `cv2.threshold`.

## 🏷️ Etiquetas

#visión-artificial #clase #operaciones-aritméticas #operaciones-lógicas #desbordamiento #normalización #python #opencv

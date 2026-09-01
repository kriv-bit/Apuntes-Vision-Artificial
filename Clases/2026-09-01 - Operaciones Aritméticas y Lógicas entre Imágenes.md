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
  - desbordamiento
  - normalización
  - python
---

# Clase 4 — Operaciones Aritméticas y Lógicas entre Imágenes

> [!info] Fecha
> 📅 Martes, 1 de septiembre de 2026 — Semana 4

## 📝 Temas vistos

1. **Requisito dimensional**: Para operar dos matrices de imagen $f_1(x, y)$ y $f_2(x, y)$, ambas deben poseer idéntica dimensionalidad ($M \times N$). La operación se realiza píxel a píxel (*point-to-point operation*):
   $$ g(x, y) = f_1(x, y) \circ f_2(x, y) $$
2. **Operaciones Aritméticas Principales**:
   - **Suma y Promedio ($f_1 + f_2$)**: Reducción de ruido aleatorio (ruido gaussiano / estocástico). Al promediar una ráfaga de $K$ capturas de una escena estática, el ruido tiende a anularse ($E[\text{ruido}] \to 0$).
   - **Resta ($|f_1 - f_2|$)**: Aislamiento de anomalías, detección de movimiento y sustracción de fondo estático.
   - **Multiplicación ($f_1 \times f_2$)**: Extracción de Regiones de Interés (ROI) mediante máscaras binarias ($0$ apaga, $1$ preserva).
   - **División ($f_1 / f_2$)**: Corrección de sombreado (*shading correction*), viñeteado óptico y no uniformidad en la iluminación.
3. **Manejo del Rango Dinámico en 8 bits (0 a 255)**:
   - **Desbordamiento Modular (*Overflow / Underflow*)**: Comportamiento nativo de enteros de 8 bits sin signo (`uint8`), donde $300 \pmod{256} = 44$.
   - **Saturación / Truncamiento (*Clipping*)**: Forzado al rango $[0, 255]$ mediante $\min(\max(val, 0), 255)$ (`cv2.add`).
   - **Normalización Min-Max (Reescalado)**: Mapeo lineal estricto sin pérdida por truncamiento.
   - **Mezcla Ponderada (*Alpha Blending*)**: $g = \alpha f_1 + \beta f_2 + \gamma$ (`cv2.addWeighted`).
4. **Álgebra de Boole sobre Imágenes Binarias**:
   - Operadores AND, OR, NOT y XOR para morfología y combinación de máscaras.

---

## 🧮 Operaciones y Ejercicios Resueltos Paso a Paso

### 📌 1. Promedio de Intensidades (Reducción de Ruido)
Supongamos la captura de una escena (ej. volcán) bajo tres condiciones de iluminación/ruido donde el mismo píxel registra valores: $4, 100, 50$.

$$ \bar{f}(x, y) = \frac{1}{K} \sum_{i=1}^K f_i(x, y) $$

**Paso a paso:**
1. Sumar los valores observados: $4 + 100 + 50 = 154$
2. Dividir entre la cantidad de capturas ($K = 3$):
$$ \bar{f}(x, y) = \frac{154}{3} \approx \mathbf{51.33} $$

---

### 📌 2. Desbordamiento Modular vs Saturación
Sean dos intensidades $p_1 = 200$ y $p_2 = 100$:

#### A. Suma aritmética teórica:
$$ 200 + 100 = 300 $$

#### B. Desbordamiento con NumPy (`uint8`):
Al exceder el límite de 255, se aplica aritmética de módulo $2^8 = 256$:
$$ 300 \pmod{256} = 300 - 256 = \mathbf{44} \quad \text{(Efecto visual: píxel blanco se torna oscuro)} $$

#### C. Suma con Saturación (`cv2.add`):
$$ \text{resultado} = \min(300, 255) = \mathbf{255} \quad \text{(Efecto visual: blanco puro sin distorsión modular)} $$

---

### 📌 3. Normalización Min-Max Paso a Paso
Dada una matriz resultante con valores fuera del rango $[0, 255]$ (o con negativos por resta):

$$ f_s = 255 \times \left[ \frac{f - \min(f)}{\max(f) - \min(f)} \right] $$

**Ejemplo numérico con una muestra de píxeles**: $F = [-50, 0, 100, 300, 460]$
- $\min(F) = -50$
- $\max(F) = 460$
- Rango dinámico: $\max(F) - \min(F) = 460 - (-50) = 510$

**Transformación de cada píxel:**
1. Para $f = -50$:
   $$ f_s = 255 \times \frac{-50 - (-50)}{510} = 255 \times \frac{0}{510} = \mathbf{0} $$
2. Para $f = 0$:
   $$ f_s = 255 \times \frac{0 - (-50)}{510} = 255 \times \frac{50}{510} = 255 \times 0.09804 \approx \mathbf{25} $$
3. Para $f = 100$:
   $$ f_s = 255 \times \frac{100 - (-50)}{510} = 255 \times \frac{150}{510} = 255 \times 0.29412 \approx \mathbf{75} $$
4. Para $f = 300$:
   $$ f_s = 255 \times \frac{300 - (-50)}{510} = 255 \times \frac{350}{510} = 255 \times 0.68627 \approx \mathbf{175} $$
5. Para $f = 460$:
   $$ f_s = 255 \times \frac{460 - (-50)}{510} = 255 \times \frac{510}{510} = 255 \times 1 = \mathbf{255} $$

---

## 🖼️ Comparaciones Visuales de Clase

### Imágenes Originales
![[operaciones-imagenes-fuente.png]]

### Overflow vs Saturación vs Normalización
![[operaciones-suma-comparacion.png]]

---

## 🔗 Notas relacionadas

- 🧠 Conceptos:
  - [[Operaciones Aritméticas entre Imágenes]]
  - [[Desbordamiento y Normalización de Imágenes]]
  - [[Operaciones Lógicas y Álgebra Booleana en Imágenes]]
  - [[Resolución de Imagen]]
- 💻 Código:
  - [[Operaciones Aritméticas y Normalización - Python]]
- 📅 Sesiones previas:
  - [[2026-08-19 - Relaciones entre Píxeles y Métricas de Distancia]]
  - [[2026-08-18 - Interpolación y Redimensionamiento]]

## 📌 Pendientes / tareas

- [ ] Implementar sustracción de fondo en video usando `cv2.absdiff`.
- [ ] Aplicar corrección de sombreado dividiendo por una imagen patrón de iluminación suave.

## 🏷️ Etiquetas

#visión-artificial #clase #operaciones-aritméticas #desbordamiento #normalización #python

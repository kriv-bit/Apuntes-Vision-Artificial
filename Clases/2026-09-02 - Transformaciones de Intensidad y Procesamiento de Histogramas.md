---
titulo: "Clase 5 — Transformaciones de Intensidad y Procesamiento de Histogramas"
materia: Visión Artificial
tipo: clase
fecha: 2026-09-02
semana: 4
tags:
  - visión-artificial
  - clase
  - transformaciones-espaciales
  - corrección-gamma
  - histogramas
  - ecualización
  - python
  - opencv
---

# Clase 5 — Transformaciones de Intensidad y Procesamiento de Histogramas

> [!info] Fecha
> 📅 Miércoles, 2 de septiembre de 2026 — Semana 4

## 📝 Temas vistos

1. **Núcleo del procesamiento en el dominio espacial (Operaciones Puntuales)**:
   - Mapeo directo píxel a píxel con ventana de vecindad $1 \times 1$:
     $$ s = T(r) $$
     donde $r$ es la intensidad de entrada del píxel y $s$ es la intensidad procesada de salida, con $r, s \in [0, L-1]$.
2. **Transformaciones de Intensidad Fundamentales**:
   - **Inversión Fotográfica / Negativo ($s = (L-1) - r$)**: Resalta estructuras claras o vasculares sobre fondos oscuros predominantes (ej. radiografías y mamografías).
   - **Transformación Logarítmica ($s = c \log(1 + r)$)**: Expande el rango dinámico de los tonos oscuros y comprime los píxeles brillantes (esencial para visualizar espectros de Fourier).
   - **Transformación de Ley de Potencias / Corrección Gamma ($s = c \cdot r^\gamma$)**: Calibración no lineal para pantallas y sensores ($\gamma < 1$ expande oscuros / aclara; $\gamma > 1$ comprime oscuros / oscurece).
   - **Estiramiento de Contraste por Tramos (*Piecewise Linear*)**: Expande un subrango estrecho de intensidades $[r_1, r_2]$ hacia todo el rango disponible $[0, 255]$.
3. **Histogramas de Imagen**:
   - Representación estadística de la distribución de niveles de gris $h(r_k) = n_k$.
   - Diagnóstico visual según el histograma: oscuro (cargado a la izquierda), brillante (cargado a la derecha), bajo contraste (pico estrecho central), alto contraste (distribución amplia en todo el rango).
4. **Ecualización de Histograma (*Histogram Equalization*)**:
   - Técnica automática que redistribuye los niveles de gris para obtener un histograma aproximadamente uniforme (plano), maximizando el contraste global mediante la Función de Distribución Acumulada (CDF).
5. **Especificación / Emparejamiento de Histograma (*Histogram Matching*)**:
   - Modificación de la imagen para que su histograma adopte una forma o distribución predefinida específica.

---

## 🔍 Corrección y Clarificación Teórica

> [!important] Aclaración sobre la Transformación Logarítmica
> - **Efecto real:** La función logaritmo **no** potencia los niveles altos; hace exactamente lo contrario:
>   - **Expande** las intensidades bajas (tonos oscuros), haciéndolas más distinguibles y brillantes.
>   - **Comprime** las intensidades altas (tonos brillantes), evitando que los picos de intensidad deslumbren u oculten el resto de la información.
> - Por ello, se usa obligatoriamente al graficar el **espectro de Fourier**, donde el componente central (DC) es millones de veces más intenso que las frecuencias altas.

---

## 🧮 Ejemplos y Operaciones Resueltas Paso a Paso

### 📌 1. Cálculo del Factor de Escala $c$ en Transformación Logarítmica
Para garantizar que la salida esté en el rango $[0, 255]$ con $r_{\max} = 255$:

$$ s = c \log(1 + r) \implies 255 = c \log(1 + 255) $$
$$ c = \frac{255}{\log(1 + 255)} = \frac{255}{\log(256)} \approx \frac{255}{5.545} \approx \mathbf{45.98} \quad (\text{usando } \ln) $$
*(O $c = \frac{255}{\log_{10}(256)} \approx \frac{255}{2.408} \approx \mathbf{105.89}$ si se usa base 10).*

---

### 📌 2. Corrección Gamma Paso a Paso
Para un píxel oscuro $r = 64$ en una escala normalizada $r_{norm} = \frac{64}{255} \approx 0.2509$:

#### A. Con $\gamma = 0.4$ (Expansión de oscuros / Aclarado):
$$ s_{norm} = (0.2509)^{0.4} \approx 0.5755 $$
$$ s = 255 \times 0.5755 \approx \mathbf{147} \quad \text{(El píxel oscuro sube de 64 a 147)} $$

#### B. Con $\gamma = 2.5$ (Compresión de oscuros / Oscurecido):
$$ s_{norm} = (0.2509)^{2.5} \approx 0.0315 $$
$$ s = 255 \times 0.0315 \approx \mathbf{8} \quad \text{(El píxel se vuelve aún más oscuro)} $$

---

### 📌 3. Estiramiento de Contraste Lineal
Dada una imagen con bajo contraste donde los píxeles útiles están concentrados entre $[r_{\min}, r_{\max}] = [50, 170]$:

$$ s = 255 \times \left( \frac{r - 50}{170 - 50} \right) = 255 \times \left( \frac{r - 50}{120} \right) $$

- Para $r = 50 \implies s = \mathbf{0}$
- Para $r = 110 \implies s = 255 \times \frac{60}{120} = \mathbf{128}$
- Para $r = 170 \implies s = \mathbf{255}$

---

### 📌 4. Demostración de Ecualización de Histograma
Para una micro-imagen de $4 \times 4$ ($N = 16$ píxeles) con $L = 8$ niveles de gris ($0$ a $7$):

| Nivel $r_k$ | Conteo $n_k$ | Probabilidad $p(r_k) = n_k/16$ | CDF Acumulada $\sum p$ | $s_k = \text{round}(7 \times \text{CDF})$ |
| :---------: | :----------: | :----------------------------: | :--------------------: | :---------------------------------------: |
| **0** | 8 | $8/16 = 0.500$ | $0.500$ | $\text{round}(7 \times 0.500) = \mathbf{4}$ |
| **1** | 4 | $4/16 = 0.250$ | $0.750$ | $\text{round}(7 \times 0.750) = \mathbf{5}$ |
| **2** | 2 | $2/16 = 0.125$ | $0.875$ | $\text{round}(7 \times 0.875) = \mathbf{6}$ |
| **3** | 2 | $2/16 = 0.125$ | $1.000$ | $\text{round}(7 \times 1.000) = \mathbf{7}$ |
| **4-7** | 0 | $0.000$ | $1.000$ | $\mathbf{7}$ |

> [!tip] Conclusión
> Los niveles originales comprimidos en el rango oscuro $[0, 3]$ fueron redistribuidos a $[4, 7]$, expandiendo el rango dinámico.

---

## 🔗 Notas relacionadas

- 🧠 Conceptos:
  - [[Transformaciones de Intensidad Espacial]]
  - [[Histogramas y Ecualización de Imagen]]
  - [[Desbordamiento y Normalización de Imágenes]]
  - [[Resolución de Imagen]]
- 💻 Código:
  - [[Transformaciones de Intensidad y Histogramas - Python]]
- 📅 Sesión previa:
  - [[2026-09-01 - Operaciones Aritméticas y Lógicas entre Imágenes]]

## 📌 Pendientes / tareas

- [ ] Implementar CLAHE (*Contrast Limited Adaptive Histogram Equalization*) para ecualización local adaptativa.
- [ ] Aplicar transformación logarítmica sobre la Transformada Rápida de Fourier (FFT) en Python con `np.fft.fft2`.

## 🏷️ Etiquetas

#visión-artificial #clase #transformaciones-espaciales #corrección-gamma #histogramas #ecualización #python #opencv

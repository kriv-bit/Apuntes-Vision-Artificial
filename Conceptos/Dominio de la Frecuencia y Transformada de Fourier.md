---
titulo: "Dominio de la Frecuencia y Transformada de Fourier"
materia: Visión Artificial
tipo: concepto
fecha: 2026-10-21
unidad: 3
tags:
  - visión-artificial
  - procesamiento-de-imágenes
  - fourier
  - dft
  - fft
  - espectro
  - escala-logarítmica
aliases:
  - Dominio de la frecuencia
  - Transformada de Fourier 2D
  - 2D-DFT
  - Espectro de Fourier
---

# Dominio de la Frecuencia y Transformada de Fourier

## 🎯 Definición

El **Dominio de la Frecuencia** representa una imagen digital como una superposición de ondas sinusoidales 2D (senos y cosenos) de diferentes frecuencias espaciales, amplitudes y orientaciones angulares.

A diferencia del dominio espacial, donde cada píxel $f(x, y)$ denota la luminosidad en una coordenada física, en el dominio frecuencial cada punto $F(u, v)$ denota la **contribución de una onda con frecuencia espacial $(u, v)$**.

---

## 🧮 De la Serie de Fourier a la Transformada 2D

### 1. La Serie de Fourier (Señales Periódicas 1D)
Toda señal periódica continua $f(t) = f(t + T)$ con frecuencia fundamental $\omega_0 = \frac{2\pi}{T}$ puede descomponerse en una suma infinita de armónicos:

$$ f(t) = a_0 + \sum_{n=1}^{\infty} \left[ a_n \cos(n \omega_0 t) + b_n \sin(n \omega_0 t) \right] $$

#### El Caso de la Onda Cuadrada y los Bordes:
Una onda cuadrada simétrica se sintetiza mediante armónicos impares:
$$ f(t) = \sum_{k=1,3,5,\dots}^{\infty} \frac{4}{k\pi} \sin(k \omega_0 t) $$
* Las **bajas frecuencias ($k=1$)** aportan la masa y forma general.
* Las **altas frecuencias ($k \gg 1$)** aportan los coeficientes decrecientes ($1/k$) que hacen vertical la transición y forman esquinas afiladas (bordes en imágenes).

### 2. Transformada Discreta de Fourier Bidimensional (2D-DFT)
Para una imagen digital de dimensiones $M \times N$:

$$ F(u, v) = \sum_{x=0}^{M-1} \sum_{y=0}^{N-1} f(x, y) e^{-j 2\pi \left( \frac{ux}{M} + \frac{vy}{N} \right)} $$

### 3. Transformada Inversa (2D-IDFT)
Permite regresar del dominio espectral al dominio espacial sin pérdida de información:

$$ f(x, y) = \frac{1}{MN} \sum_{u=0}^{M-1} \sum_{v=0}^{N-1} F(u, v) e^{j 2\pi \left( \frac{ux}{M} + \frac{vy}{N} \right)} $$

---

## 🔬 El Componente DC y la Escala Logarítmica

En el origen de frecuencias $(u=0, v=0)$:
$$ F(0, 0) = \sum_{x=0}^{M-1} \sum_{y=0}^{N-1} f(x, y) = M \times N \times \bar{f} $$
Este valor se denomina **Componente DC (*Direct Current*)** y equivale a la suma total de intensidad de la imagen (directamente proporcional al brillo medio).

### ¿Por qué es obligatoria la transformación logarítmica?
El componente DC suele ser de orden $10^6 - 10^8$, mientras que los coeficientes de alta frecuencia suelen ser del orden de $10^0 - 10^2$. Si se escala linealmente a una pantalla de 8 bits $[0, 255]$:
* $F(0, 0)$ toma el valor 255 (blanco).
* Casi todo el resto de la matriz queda por debajo de 0.5 (negro puro).
* El ojo humano solo vería un punto blanco aislado.

Para hacer visible todo el espectro armónico, se comprime el rango dinámico mediante la **transformación logarítmica**:
$$ D(u, v) = c \cdot \log(1 + |F(u, v)|) $$

---

## 🔄 El Centrado Espectral (`fftshift`)

Por definición matemática, la DFT ubica el componente de frecuencia cero $(0, 0)$ en la esquina superior izquierda, y las frecuencias altas en el interior.

Para facilitar la interpretación visual y el diseño de filtros simétricos, se realiza un desplazamiento de cuadrantes (**`np.fft.fftshift`**):
* Intercambia los cuadrantes diagonales ($1 \leftrightarrow 3$ y $2 \leftrightarrow 4$).
* Ubica el componente DC $(0, 0)$ en el **centro geométrico** de la matriz:
  - **Centro:** Bajas frecuencias (fondos y formas globales).
  - **Bordes exteriores:** Altas frecuencias (aristas, texturas y ruido).
  - **Distancia radial $r = \sqrt{u^2 + v^2}$:** Frecuencia espacial euclidiana.

---

## 🔗 Relacionado

- [[Magnitud y Fase en Fourier]]
- [[Filtrado en el Dominio de la Frecuencia]]
- [[2026-10-21 - Introducción al Dominio de la Frecuencia y Transformada de Fourier]]
- [[Transformada de Fourier y Filtrado en Frecuencia - Python]]

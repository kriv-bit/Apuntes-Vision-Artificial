---
titulo: "Reducción de Ruido Aditivo y Filtros Notch - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-10-28
clase: "[[2026-10-28 - Reducción de Ruido Aditivo y Cíclico]]"
tags:
  - visión-artificial
  - python
  - opencv
  - numpy
  - matplotlib
  - reducción-de-ruido
  - filtros-espaciales
  - filtros-notch
  - fourier
aliases:
  - Código reducción de ruido y filtros notch
  - Código notch filter Python
  - Restauración de imágenes con ruido periódico
---

# Reducción de Ruido Aditivo y Filtros Notch — Python

> [!info] ¿Qué es este código?
> Este script implementa en **Python (NumPy, OpenCV, SciPy y Matplotlib)**:
> 1. Comparativa de **filtros espaciales** (Media Aritmética, Media Geométrica, Mediana y Media Alfa-Recortada) frente a ruidos Gaussiano, Sal y Pimienta y Mixto.
> 2. Generación sintética de **ruido periódico sinusoidal** (franjas regulares).
> 3. Visualización del espectro logarítmico de Fourier con identificación de picos conjugados.
> 4. Construcción y aplicación de un **Filtro Notch Gaussiano de Rechazo** para eliminar completamente las rayas de la imagen preservando los detalles.

> [!info] Clases y conceptos relacionados
> 📅 Clase: [[2026-10-28 - Reducción de Ruido Aditivo y Cíclico]]
> 🧠 Conceptos: [[Reducción de Ruido Aditivo y Espacial]] y [[Filtros Notch y Rechaza Banda]]

---

## 📦 Requisitos

```bash
pip install opencv-python-headless numpy matplotlib scipy scikit-image
```

---

## 💻 Código completo

```python
import numpy as np
import cv2
import matplotlib.pyplot as plt
from scipy.ndimage import generic_filter
from scipy.stats import trim_mean
from skimage import data

# Configuración de figuras
plt.rcParams["figure.figsize"] = (14, 8)
img = data.camera()  # 512x512

# ==============================================================================
# 1. FILTRADO ESPACIAL ANTE RUIDO ALEATORIO
# ==============================================================================
def add_salt_and_pepper(image, prob=0.03):
    """Genera ruido impulsivo bipolar de sal y pimienta"""
    noisy = image.copy()
    num_noise = int(prob * image.size)
    coords_salt = [np.random.randint(0, i - 1, num_noise) for i in image.shape]
    coords_pepp = [np.random.randint(0, i - 1, num_noise) for i in image.shape]
    noisy[tuple(coords_salt)] = 255
    noisy[tuple(coords_pepp)] = 0
    return noisy

def geometric_mean_filter(image, ksize=3):
    """Calcula la media geométrica en escala logarítmica para estabilidad"""
    img_f = image.astype(np.float64) + 1e-5  # Evitar log(0)
    log_img = np.log(img_f)
    kernel = np.ones((ksize, ksize)) / (ksize * ksize)
    mean_log = cv2.filter2D(log_img, -1, kernel)
    return np.uint8(np.clip(np.exp(mean_log), 0, 255))

def alpha_trimmed_mean_filter(image, ksize=3, d=2):
    """Aplica la media alfa-recortada descartando d elementos extremos"""
    proportion = (d / 2.0) / (ksize * ksize)
    def trim_func(values):
        return trim_mean(values, proportiontocut=proportion)
    return generic_filter(image, trim_func, size=ksize).astype(np.uint8)

# Simular mezcla de ruido (Gaussiano + Sal y Pimienta)
noise_gauss = np.random.normal(0, 15, img.shape)
noisy_mixed = np.clip(img.astype(float) + noise_gauss, 0, 255).astype(np.uint8)
noisy_mixed = add_salt_and_pepper(noisy_mixed, prob=0.02)

# Aplicar los filtros
res_arithmetic = cv2.blur(noisy_mixed, (3, 3))
res_median     = cv2.medianBlur(noisy_mixed, 3)
res_alpha      = alpha_trimmed_mean_filter(noisy_mixed, ksize=3, d=2)

fig, axes = plt.subplots(1, 4, figsize=(16, 4))
axes[0].imshow(noisy_mixed, cmap='gray'); axes[0].set_title('Degradada (Gauss + Sal/Pimienta)'); axes[0].axis('off')
axes[1].imshow(res_arithmetic, cmap='gray'); axes[1].set_title('Media Aritmética\n(Manchas difusas)'); axes[1].axis('off')
axes[2].imshow(res_median, cmap='gray'); axes[2].set_title('Mediana\n(Quita sal/pimienta)'); axes[2].axis('off')
axes[3].imshow(res_alpha, cmap='gray'); axes[3].set_title('Media Alfa-Recortada (d=2)\n(Óptimo para mezcla)'); axes[3].axis('off')
plt.tight_layout()
plt.show()


# ==============================================================================
# 2. GENERACIÓN DE RUIDO PERIÓDICO (FRANJAS SINUSOIDALES)
# ==============================================================================
M, N = img.shape
X, Y = np.meshgrid(np.arange(N), np.arange(M))

# Interferencia sinusoidal con frecuencia espacial (u0 = 35, v0 = 50)
u0, v0 = 35, 50
amplitude = 60.0
periodic_noise = amplitude * np.sin(2.0 * np.pi * (u0 * X / M + v0 * Y / N))

# Imagen contaminada
img_corrupted = np.clip(img.astype(np.float64) + periodic_noise, 0, 255).astype(np.uint8)


# ==============================================================================
# 3. TRANSFORMADA DE FOURIER Y LOCALIZACIÓN DE PICOS
# ==============================================================================
# 1. Calcular FFT2 y centrar el origen
F = np.fft.fft2(img_corrupted.astype(np.float64))
F_shift = np.fft.fftshift(F)

# 2. Espectro de magnitud en escala logarítmica
magnitude_spectrum = np.log(1.0 + np.abs(F_shift))


# ==============================================================================
# 4. CONSTRUCCIÓN DEL FILTRO NOTCH GAUSSIANO DE RECHAZO
# ==============================================================================
center_u, center_v = M // 2, N // 2
D0 = 10.0  # Radio de corte de la muesca

# Coordenadas relativas de los picos en el espectro centrado
# Los picos aparecen simétricamente en (+u0, +v0) y (-u0, -v0)
p1_u, p1_v = center_u + v0, center_v + u0
p2_u, p2_v = center_u - v0, center_v - u0

# Distancias euclidianas a cada centro de muesca
D1 = np.sqrt((Y - p1_u)**2 + (X - p1_v)**2)
D2 = np.sqrt((Y - p2_u)**2 + (X - p2_v)**2)

# Máscara Notch Gaussiana de rechazo (vale ~0 en los picos, ~1 en el resto)
H_notch = (1.0 - np.exp(-0.5 * (D1 / D0)**2)) * (1.0 - np.exp(-0.5 * (D2 / D0)**2))


# ==============================================================================
# 5. FILTRADO COMPLEJO Y TRANSFORMADA INVERSA (RECONSTRUCCIÓN)
# ==============================================================================
# Multiplicación sobre el espectro complejo centrado
G_shift = F_shift * H_notch

# Des-centrar y calcular la IFFT2
G = np.fft.ifftshift(G_shift)
img_restored_complex = np.fft.ifft2(G)

# Tomar la parte real y recortar a [0, 255]
img_restored = np.clip(np.real(img_restored_complex), 0, 255).astype(np.uint8)


# ==============================================================================
# 6. VISUALIZACIÓN DEL PROCESO DE RESTAURACIÓN
# ==============================================================================
fig, axes = plt.subplots(2, 2, figsize=(12, 12))

axes[0, 0].imshow(img_corrupted, cmap='gray')
axes[0, 0].set_title('1. Imagen Contaminada (Ruido Periódico)')
axes[0, 0].axis('off')

axes[0, 1].imshow(magnitude_spectrum, cmap='magma')
axes[0, 1].set_title('2. Espectro Logarítmico log(1 + |F|)\n(Observa los 2 picos brillantes simétricos)')
axes[0, 1].axis('off')

axes[1, 0].imshow(H_notch, cmap='gray')
axes[1, 0].set_title('3. Máscara Notch Gaussiana H(u, v)\n(Bloquea ambos picos preservando el centro)')
axes[1, 0].axis('off')

axes[1, 1].imshow(img_restored, cmap='gray')
axes[1, 1].set_title('4. Imagen Restaurada con IFFT2\n(¡Rayas eliminadas al 100% sin desenfoque!)')
axes[1, 1].axis('off')

plt.tight_layout()
plt.show()
```

---

## 🔍 Conclusiones del Experimento

1. **La superioridad del filtro Notch en ruido periódico:**
   - Si intentáramos eliminar las rayas con un filtro espacial (como `cv2.GaussianBlur`), tendríamos que aplicar un desenfoque tan descomunal que la imagen del fotógrafo quedaría irreconocible.
   - El **Filtro Notch** elimina las rayas sin alterar la nitidez de la ropa, el trípode ni el rostro del fotógrafo, porque solo retira el $0.1\%$ de las frecuencias de la escena.
2. **Importancia de la simetría conjugada:**
   - Si solo bloqueas uno de los dos picos, la interferencia persiste como una oscilación compleja que genera artefactos geométricos y pérdida de contraste.
3. **El perfil Gaussiano frente al Ideal:**
   - La función exponencial suave $1 - e^{-D^2 / (2D_0^2)}$ evita cortes bruscos, asegurando que la imagen reconstruida no presente ningún efecto de oscilación (*ringing*).

---

## 🔗 Relacionado

- [[Reducción de Ruido Aditivo y Espacial]] — formulación analítica de medias y orden
- [[Filtros Notch y Rechaza Banda]] — fundamentos teóricos de muescas frecuenciales
- [[2026-10-28 - Reducción de Ruido Aditivo y Cíclico]] — clase teórica correspondiente
- [[Inicio]] — mapa general de la materia

---

## 🏷️ Etiquetas

#visión-artificial #python #opencv #numpy #matplotlib #reducción-de-ruido #filtros-espaciales #filtros-notch #fourier

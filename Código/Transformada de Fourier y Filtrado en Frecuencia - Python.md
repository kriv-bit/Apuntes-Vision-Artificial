---
titulo: "Transformada de Fourier y Filtrado en Frecuencia - Python"
materia: Visión Artificial
tipo: codigo
fecha: 2026-10-21
unidad: 3
tags:
  - visión-artificial
  - opencv
  - python
  - fourier
  - fft2
  - ifft2
  - filtrado-en-frecuencia
  - notch
aliases:
  - Fourier en Python
  - FFT en OpenCV
  - Filtrado en frecuencia con NumPy
---

# Transformada de Fourier y Filtrado en Frecuencia — Python

Guía paso a paso para calcular la Transformada de Fourier 2D, visualizar el espectro en escala logarítmica, manipular magnitud y fase, y aplicar filtros pasa-bajas, pasa-altas y notch en Python con NumPy y OpenCV.

---

## 🐍 1. Cálculo de la FFT y Visualización Logarítmica

```python
import cv2
import numpy as np
import matplotlib.pyplot as plt
from skimage import data

# 1. Cargar imagen en escala de grises
img = data.camera()

# 2. Calcular la FFT 2D y centrarla en el origen
fft = np.fft.fft2(img)
fft_shift = np.fft.fftshift(fft)

# 3. Magnitud en escala lineal vs. logarítmica
magnitude = np.abs(fft_shift)
magnitude_log = np.log(1 + magnitude)

# Visualización
fig, axes = plt.subplots(1, 3, figsize=(16, 5))

axes[0].imshow(img, cmap='gray')
axes[0].set_title("1. Imagen Original (Espacio)", fontweight='bold')
axes[0].axis('off')

axes[1].imshow(magnitude, cmap='gray')
axes[1].set_title("2. Espectro Lineal\n(Solo se ve el punto central DC)", fontweight='bold')
axes[1].axis('off')

axes[2].imshow(magnitude_log, cmap='gray')
axes[2].set_title("3. Espectro Logarítmico $\log(1 + |F|)$\n(Altas frecuencias visibles)", fontweight='bold', color='darkblue')
axes[2].axis('off')

plt.tight_layout()
plt.show()
```

---

## 🐍 2. Experimento de Reconstrucción: Magnitud vs. Fase

```python
# Cargar dos imágenes distintas para cruce
img_camera = data.camera()
img_coins = cv2.resize(data.coins(), img_camera.shape[::-1])

# Calcular FFT de ambas
F_cam = np.fft.fft2(img_camera)
F_coi = np.fft.fft2(img_coins)

# Extraer magnitud y fase
mag_cam, phase_cam = np.abs(F_cam), np.angle(F_cam)
mag_coi, phase_coi = np.abs(F_coi), np.angle(F_coi)

# Cruce: Magnitud de Monedas + Fase de Camera
F_hybrid_1 = mag_coi * np.exp(1j * phase_cam)
reconstructed_1 = np.real(np.fft.ifft2(F_hybrid_1))

# Cruce: Magnitud de Camera + Fase de Monedas
F_hybrid_2 = mag_cam * np.exp(1j * phase_coi)
reconstructed_2 = np.real(np.fft.ifft2(F_hybrid_2))

# Visualización
fig, axes = plt.subplots(1, 4, figsize=(18, 5))
axes[0].imshow(img_camera, cmap='gray')
axes[0].set_title("Imagen A: Cámara")
axes[0].axis('off')

axes[1].imshow(img_coins, cmap='gray')
axes[1].set_title("Imagen B: Monedas")
axes[1].axis('off')

axes[2].imshow(reconstructed_1, cmap='gray')
axes[2].set_title("Mag(Monedas) + Fase(Cámara)\n-> ¡Se ve la Cámara!", fontweight='bold', color='darkgreen')
axes[2].axis('off')

axes[3].imshow(reconstructed_2, cmap='gray')
axes[3].set_title("Mag(Cámara) + Fase(Monedas)\n-> ¡Se ven las Monedas!", fontweight='bold', color='darkblue')
axes[3].axis('off')

plt.tight_layout()
plt.show()
```

---

## 🐍 3. Filtros Pasa-Bajas y Pasa-Altas Gaussianos

```python
def create_gaussian_filters(shape, cutoff=30):
    M, N = shape
    u = np.arange(M) - M // 2
    v = np.arange(N) - N // 2
    U, V = np.meshgrid(v, u)
    D = np.sqrt(U**2 + V**2)
    
    # Filtro Pasa-Bajas Gaussiano (GLPF)
    H_low = np.exp(-(D**2) / (2 * (cutoff**2)))
    # Filtro Pasa-Altas Gaussiano (GHPF)
    H_high = 1.0 - H_low
    return H_low, H_high

H_low, H_high = create_gaussian_filters(img.shape, cutoff=35)

# Aplicar en frecuencia (multiplicación punto a punto)
G_low = fft_shift * H_low
G_high = fft_shift * H_high

# Transformada Inversa
img_low = np.real(np.fft.ifft2(np.fft.ifftshift(G_low)))
img_high = np.real(np.fft.ifft2(np.fft.ifftshift(G_high)))

# Visualización
fig, axes = plt.subplots(1, 4, figsize=(18, 4.5))
axes[0].imshow(img, cmap='gray')
axes[0].set_title("Original")
axes[0].axis('off')

axes[1].imshow(H_low, cmap='gray')
axes[1].set_title("Máscara Pasa-Bajas (GLPF)")
axes[1].axis('off')

axes[2].imshow(img_low, cmap='gray')
axes[2].set_title("Resultado Pasa-Bajas\n(Suavizado / Blur)", fontweight='bold')
axes[2].axis('off')

axes[3].imshow(img_high, cmap='gray')
axes[3].set_title("Resultado Pasa-Altas\n(Extracción de Bordes)", fontweight='bold')
axes[3].axis('off')

plt.tight_layout()
plt.show()
```

---

## 🐍 4. Eliminación de Ruido Periódico con Filtro Notch (Muesca)

```python
# 1. Simular ruido sinusoidal periódico en la imagen
M, N = img.shape
x = np.arange(N)
y = np.arange(M)
X, Y = np.meshgrid(x, y)
# Ruido diagonal de alta frecuencia
noise_sin = 30 * np.sin(2 * np.pi * (X * 0.15 + Y * 0.15))
img_corrupted = np.clip(img.astype(np.float64) + noise_sin, 0, 255).astype(np.uint8)

# 2. FFT de la imagen con ruido
F_noisy = np.fft.fftshift(np.fft.fft2(img_corrupted))
spectrum_noisy = np.log(1 + np.abs(F_noisy))

# 3. Diseñar filtro Notch: anular los puntos simétricos del ruido
H_notch = np.ones((M, N), dtype=np.float64)
# Coordenadas relativas del ruido sinusoidal (simétricas)
center_y, center_x = M // 2, N // 2
u_offset, v_offset = int(M * 0.15), int(N * 0.15)
radius = 6

cv2.circle(H_notch, (center_x + v_offset, center_y + u_offset), radius, 0, -1)
cv2.circle(H_notch, (center_x - v_offset, center_y - u_offset), radius, 0, -1)

# 4. Filtrar y recuperar
F_filtered = F_noisy * H_notch
img_restored = np.real(np.fft.ifft2(np.fft.ifftshift(F_filtered)))

# Visualización
fig, axes = plt.subplots(1, 4, figsize=(18, 4.5))
axes[0].imshow(img_corrupted, cmap='gray')
axes[0].set_title("1. Imagen con Ruido Periódico")
axes[0].axis('off')

axes[1].imshow(spectrum_noisy, cmap='gray')
axes[1].set_title("2. Espectro con Puntos de Ruido")
axes[1].axis('off')

axes[2].imshow(H_notch, cmap='gray')
axes[2].set_title("3. Filtro Notch (Muesca)")
axes[2].axis('off')

axes[3].imshow(img_restored, cmap='gray')
axes[3].set_title("4. Imagen Restaurada Sin Ruido", fontweight='bold', color='darkgreen')
axes[3].axis('off')

plt.tight_layout()
plt.show()
```

---

## 🔗 Relacionado

- [[Dominio de la Frecuencia y Transformada de Fourier]]
- [[Magnitud y Fase en Fourier]]
- [[Filtrado en el Dominio de la Frecuencia]]
- [[2026-10-21 - Introducción al Dominio de la Frecuencia y Transformada de Fourier]]

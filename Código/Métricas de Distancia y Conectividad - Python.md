---
titulo: "Métricas de Distancia y Conectividad - Python"
materia: Visión Artificial
tipo: código
fecha: 2026-08-19
clase: "[[2026-08-19 - Relaciones entre Píxeles y Métricas de Distancia]]"
tags:
  - visión-artificial
  - python
  - numpy
  - matplotlib
  - distancias
  - conectividad
aliases:
  - Código cálculo de distancias De D4 D8
  - Código conectividad de píxeles
---

# Métricas de Distancia y Conectividad — Python

> [!info] ¿Qué es este código?
> Este script implementa las 3 funciones métricas de distancia en imágenes ($D_e$, $D_4$ y $D_8$), resuelve automáticamente los ejercicios numéricos vistos en clase y genera mapas de calor para visualizar las formas geométricas (círculo, rombo y cuadrado) de cada métrica.

> [!info] Clase y conceptos relacionados
> 📅 Clase: [[2026-08-19 - Relaciones entre Píxeles y Métricas de Distancia]]
> 🧠 Conceptos: [[Métricas de Distancia en Imágenes]] y [[Vecindad y Adyacencia de Píxeles]]

---

## 🎯 Qué hace el programa

1. Define funciones vectorizadas para calcular:
   - $D_e(p, q) = \sqrt{(x-s)^2 + (y-t)^2}$ (Euclidiana)
   - $D_4(p, q) = |x-s| + |y-t|$ (City-Block / Manhattan)
   - $D_8(p, q) = \max(|x-s|, |y-t|)$ (Chessboard / Ajedrez)
2. Resuelve y valida los dos ejercicios de clase:
   - Caso 1: $p(1,1) \to q(4,5)$
   - Caso 2: $p(1,1) \to q(5,5)$
3. Genera una cuadrícula $21 \times 21$ y grafica las isolíneas de distancia desde el centro $(10, 10)$ con Matplotlib.

---

## 📦 Requisitos

- Python 3 con `numpy`, `matplotlib`, `opencv-python`.

```bash
pip install numpy matplotlib opencv-python
```

---

## 💻 Código completo

```python
import numpy as np
import matplotlib.pyplot as plt

# ==============================================================================
# 1. DEFINICIÓN DE LAS FUNCIONES MÉTRICAS
# ==============================================================================
def distancia_euclidiana(p, q):
    """Calcula De entre p(x,y) y q(s,t)"""
    return np.sqrt((p[0] - q[0])**2 + (p[1] - q[1])**2)

def distancia_cityblock(p, q):
    """Calcula D4 (Manhattan) entre p(x,y) y q(s,t)"""
    return abs(p[0] - q[0]) + abs(p[1] - q[1])

def distancia_chessboard(p, q):
    """Calcula D8 (Chessboard/Chebyshev) entre p(x,y) y q(s,t)"""
    return max(abs(p[0] - q[0]), abs(p[1] - q[1]))

# ==============================================================================
# 2. RESOLUCIÓN DE LOS EJEMPLOS DE CLASE
# ==============================================================================
p1, q1 = (1, 1), (4, 5)
print("=== EJERCICIO 1: p(1,1) a q(4,5) ===")
print(f"De: {distancia_euclidiana(p1, q1):.2f}")     # Esperado: 5.00
print(f"D4: {distancia_cityblock(p1, q1)}")         # Esperado: 7
print(f"D8: {distancia_chessboard(p1, q1)}")        # Esperado: 4

p2, q2 = (1, 1), (5, 5)
print("\n=== EJERCICIO 2: p(1,1) a q(5,5) ===")
print(f"De: {distancia_euclidiana(p2, q2):.2f}")     # Esperado: 5.66 (sqrt(32))
print(f"D4: {distancia_cityblock(p2, q2)}")         # Esperado: 8
print(f"D8: {distancia_chessboard(p2, q2)}")        # Esperado: 4

# ==============================================================================
# 3. VISUALIZACIÓN DE ISOLÍNEAS EN UNA CUADRÍCULA
# ==============================================================================
grid_size = 21
center = (10, 10)

mat_de = np.zeros((grid_size, grid_size))
mat_d4 = np.zeros((grid_size, grid_size))
mat_d8 = np.zeros((grid_size, grid_size))

for x in range(grid_size):
    for y in range(grid_size):
        pt = (x, y)
        mat_de[x, y] = distancia_euclidiana(center, pt)
        mat_d4[x, y] = distancia_cityblock(center, pt)
        mat_d8[x, y] = distancia_chessboard(center, pt)

fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# Euclidiana
im0 = axes[0].imshow(mat_de, cmap='viridis')
axes[0].set_title("Distancia Euclidiana ($D_e$)\nIsolíneas circulares")
axes[0].plot(center[1], center[0], 'ro')
fig.colorbar(im0, ax=axes[0], fraction=0.046, pad=0.04)

# City-Block
im1 = axes[1].imshow(mat_d4, cmap='plasma')
axes[1].set_title("Distancia City-Block ($D_4$)\nIsolíneas romboidales")
axes[1].plot(center[1], center[0], 'ro')
fig.colorbar(im1, ax=axes[1], fraction=0.046, pad=0.04)

# Chessboard
im2 = axes[2].imshow(mat_d8, cmap='inferno')
axes[2].set_title("Distancia Chessboard ($D_8$)\nIsolíneas cuadradas")
axes[2].plot(center[1], center[0], 'ro')
fig.colorbar(im2, ax=axes[2], fraction=0.046, pad=0.04)

plt.tight_layout()
plt.show()
```

---

## 🔍 Explicación paso a paso

1. **Cálculo exacto**:
   - Para $p(1,1)$ y $q(4,5)$, las diferencias ortogonales son $\Delta x = 3$ y $\Delta y = 4$.
   - Al ser una terna pitagórica ($3, 4, 5$), la distancia euclidiana es exactamente $\sqrt{3^2 + 4^2} = 5.0$.
2. **Propiedad de orden**:
   - En todos los casos se comprueba: $D_8 \le D_e \le D_4$.
3. **Forma geométrica de los mapas**:
   - $D_e$ propaga círculos perfectos.
   - $D_4$ forma diamantes/rombos debido a la restricción ortogonal.
   - $D_8$ forma cuadrados perfectos porque los pasos diagonales tienen costo 1.

---

## 🔗 Relacionado

- [[Métricas de Distancia en Imágenes]] — formulación teórica
- [[Vecindad y Adyacencia de Píxeles]] — teoría de conectividad
- [[2026-08-19 - Relaciones entre Píxeles y Métricas de Distancia]] — apuntes de clase
- [[Inicio]] — mapa de contenidos

---

## 🏷️ Etiquetas

#visión-artificial #python #numpy #matplotlib #distancias #conectividad

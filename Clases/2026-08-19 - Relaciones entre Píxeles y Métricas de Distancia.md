---
titulo: "Clase 3 — Relaciones entre Píxeles y Métricas de Distancia"
materia: Visión Artificial
tipo: clase
fecha: 2026-08-19
semana: 2
tags:
  - visión-artificial
  - clase
  - vecindad
  - adyacencia
  - distancias
---

# Clase 3 — Relaciones entre Píxeles y Métricas de Distancia

> [!info] Fecha
> 📅 Miércoles, 19 de agosto de 2026 — Semana 2

## 📝 Temas vistos

1. **Vecindades de un píxel $p(x, y)$**:
   - **Vecinos-4 ($N_4(p)$)**: Desplazamientos ortogonales (arriba, abajo, izquierda, derecha).
   - **Vecinos Diagonales ($N_D(p)$ o $N_0(p)$)**: Las 4 esquinas diagonales.
   - **Vecinos-8 ($N_8(p)$)**: Unión total $N_4(p) \cup N_D(p)$.
   - Comportamiento en bordes y esquinas de una matriz.
2. **Adyacencia y Conectividad (Criterio del conjunto $V$)**:
   - Proximidad espacial + similitud de valor (intensidad dentro del conjunto $V$).
   - **4-adyacencia** y **8-adyacencia**.
   - **$m$-adyacencia (adyacencia mixta)**: Eliminación de ambigüedades y caminos redundantes.
3. **Caminos (*Paths*) y Regiones Conexas**:
   - Secuencias de píxeles conectados y definición de fronteras de objetos (típicamente evaluadas con 8-conectividad).
4. **Métricas de Distancia**:
   - Distancia Euclidiana ($D_e$).
   - Distancia City-Block / Manhattan ($D_4$).
   - Distancia Chessboard / Tablero de Ajedrez ($D_8$ o $D_s$).

---

## 🧮 Ejercicios y Operaciones Resueltas Paso a Paso

### 📌 Ejercicio 1: Distancias entre $p(1,1)$ y $q(4,5)$

Sean las coordenadas de los puntos:
- $p = (x, y) = (1, 1)$
- $q = (s, t) = (4, 5)$

Diferencias en cada eje:
- $\Delta x = |x - s| = |1 - 4| = |-3| = 3$
- $\Delta y = |y - t| = |1 - 5| = |-4| = 4$

#### 1. Distancia Euclidiana ($D_e$)
$$ D_e(p, q) = \sqrt{(x - s)^2 + (y - t)^2} $$

**Paso a paso:**
1. Reemplazar valores: $D_e = \sqrt{(1 - 4)^2 + (1 - 5)^2}$
2. Elevar al cuadrado: $D_e = \sqrt{(-3)^2 + (-4)^2} = \sqrt{9 + 16}$
3. Sumar términos: $D_e = \sqrt{25}$
4. Raíz cuadrada:
$$ \mathbf{D_e = 5} $$

#### 2. Distancia City-Block / Manhattan ($D_4$)
$$ D_4(p, q) = |x - s| + |y - t| $$

**Paso a paso:**
1. Reemplazar valores absolutos: $D_4 = |1 - 4| + |1 - 5|$
2. Calcular distancias ortogonales: $D_4 = 3 + 4$
3. Sumar:
$$ \mathbf{D_4 = 7} $$

#### 3. Distancia Chessboard / Ajedrez ($D_8$ o $D_s$)
$$ D_8(p, q) = \max(|x - s|, |y - t|) $$

**Paso a paso:**
1. Reemplazar valores absolutos: $D_8 = \max(|1 - 4|, |1 - 5|)$
2. Evaluar el máximo: $D_8 = \max(3, 4)$
3. Resultado:
$$ \mathbf{D_8 = 4} $$

---

### 📌 Ejercicio 2: Distancias entre $p(1,1)$ y $q(5,5)$ (Caso de Matriz)

Sean las coordenadas:
- $p = (x, y) = (1, 1)$
- $q = (s, t) = (5, 5)$
- $\Delta = |x - s| = |y - t| = |1 - 5| = 4$

![[metricas-distancia-matriz.png]]

#### 1. Distancia Euclidiana ($D_e$)
$$ D_e = \sqrt{(1 - 5)^2 + (1 - 5)^2} = \sqrt{(-4)^2 + (-4)^2} = \sqrt{16 + 16} = \sqrt{32} \approx \mathbf{5.66} $$

#### 2. Distancia City-Block ($D_4$)
$$ D_4 = |1 - 5| + |1 - 5| = 4 + 4 = \mathbf{8} $$

#### 3. Distancia Chessboard ($D_8$ / $D_s$)
$$ D_8 = \max(|1 - 5|, |1 - 5|) = \max(4, 4) = \mathbf{4} $$

---

## 🖼️ Ejemplos visuales de Adyacencia

![[adyacencia-y-vecinos-ejemplo.png]]

> [!note] Análisis de la imagen
> Con un conjunto de intensidades de interés $V = \{1\}$:
> - El píxel central (valor 2) **no** pertenece a $V$, pero sus vecinos inferiores sí tienen valor 1.
> - Si evaluamos un píxel con valor 1 en una esquina que solo tiene ceros y dos a su alrededor, **no tiene vecinos adyacentes** bajo la condición $V=\{1\}$.

---

## 🔗 Notas relacionadas

- 🧠 Conceptos:
  - [[Vecindad y Adyacencia de Píxeles]]
  - [[Métricas de Distancia en Imágenes]]
  - [[Resolución de Imagen]]
- 💻 Código:
  - [[Métricas de Distancia y Conectividad - Python]]
- 📅 Sesiones previas:
  - [[2026-08-18 - Interpolación y Redimensionamiento]]
  - [[2026-08-12 - Fundamentos de Imágenes]]

## 📌 Pendientes / tareas

- [ ] Implementar una función en Python que calcule la matriz de distancias $D_4$ y $D_8$ desde un punto central.
- [ ] Analizar casos prácticos donde la $m$-adyacencia previene bucles en algoritmos de adelgazamiento de bordes (*thinning/skeletonization*).

## 🏷️ Etiquetas

#visión-artificial #clase #vecindad #adyacencia #distancias

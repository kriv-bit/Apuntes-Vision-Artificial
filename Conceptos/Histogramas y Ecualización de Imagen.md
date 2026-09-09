---
titulo: "Histogramas y Ecualización de Imagen"
materia: Visión Artificial
tipo: concepto
fecha: 2026-09-02
tags:
  - visión-artificial
  - procesamiento-de-imágenes
  - histogramas
  - ecualización
  - contraste
  - histogram-matching
aliases:
  - Histograma de imagen
  - Ecualización de histograma
  - Especificación de histograma
  - Histogram matching
---

# Histogramas y Ecualización de Imagen

## 🎯 Definición

El **histograma** de una imagen digital en escala de grises con niveles de intensidad en el intervalo $[0, L-1]$ es una función discreta $h(r_k)$ que contabiliza el número de píxeles $n_k$ con intensidad $r_k$:

$$ h(r_k) = n_k \quad \text{para } k = 0, 1, 2, \dots, L-1 $$

El **histograma normalizado** representa la probabilidad empírica de ocurrencia de cada nivel de gris $p(r_k)$ en una imagen de dimensiones $M \times N$:

$$ p(r_k) = \frac{n_k}{M \cdot N} \quad \text{donde } \sum_{k=0}^{L-1} p(r_k) = 1 $$

---

## 📊 Diagnóstico Visual a través del Histograma

```mermaid
graph LR
    H[Forma del Histograma] --> Osc["Concentrado a la izquierda (bajas intensidades)"]
    H --> Bri["Concentrado a la derecha (altas intensidades)"]
    H --> Bajo["Pico estrecho en el centro"]
    H --> Alto["Distribución amplia y uniforme en todo el eje"]
    
    Osc --> Im1["Imagen Subexpuesta / Muy Oscura"]
    Bri --> Im2["Imagen Sobreexpuesta / Muy Clara"]
    Bajo --> Im3["Imagen Lavada / Bajo Contraste"]
    Alto --> Im4["Imagen Nítida / Alto Contraste"]
```

---

## 🧮 Ecualización de Histograma (*Histogram Equalization*)

La ecualización es una transformación que busca que todos los niveles de intensidad tengan igual probabilidad de aparición (histograma plano / uniforme), maximizando automáticamente el contraste global.

### Fundamento Matemático (CDF Discreta):
Para cada nivel de gris de entrada $r_k$, el nuevo nivel de salida $s_k$ se obtiene evaluando la **Función de Distribución Acumulada (CDF)**:

$$ s_k = T(r_k) = \text{round}\left( (L - 1) \sum_{j=0}^k p(r_j) \right) = \text{round}\left( \frac{L - 1}{M \cdot N} \sum_{j=0}^k n_j \right) $$

> [!important] El efecto "Peine" (*Comb Effect*)
> En el mundo discreto, la ecualización no produce una línea plana horizontal perfecta, sino barras separadas por huecos vacíos. Esto se debe a que los niveles de intensidad son números enteros finitos: no se pueden fraccionar píxeles para rellenar los huecos intermedios.

![[histograma-ecualizacion-luna-comparacion.png]]

---

## 🎯 Especificación de Histograma (*Histogram Matching*)

Mientras que la ecualización transforma la imagen para que su histograma sea siempre uniforme, la **especificación de histograma** permite forzar a la imagen a adoptar una **forma o distribución objetivo personalizada** $p_z(z)$ proveniente de una **imagen de referencia**:

```mermaid
graph LR
    R["Imagen Entrada r"] --> T["Ecualizar: s = T(r)"]
    Z["Imagen Referencia z"] --> G["Ecualizar: v = G(z)"]
    T --> INV["Mapeo Inverso: z = G⁻¹(s)"]
    G --> INV
    INV --> RES["Imagen con distribución de referencia"]
```

1. Se ecualiza la imagen original: $s = T(r)$.
2. Se calcula la transformación de ecualización para la referencia: $v = G(z)$.
3. Se aplica el mapeo inverso: $z = G^{-1}(s) = G^{-1}(T(r))$.

![[histograma-matching-especificacion-comparacion.png]]

- **Ventajas prácticas:** 
  - Normalización fotométrica entre múltiples cámaras de seguridad.
  - Corrección de cambios drásticos de iluminación en secuencias de video.
  - Igualación de contraste en imágenes médicas tomadas por distintos escáneres.

---

## 🔗 Relacionado

- [[Transformaciones de Intensidad Espacial]] — transformaciones lineales y no lineales
- [[Desbordamiento y Normalización de Imágenes]] — normalización del rango dinámico
- [[2026-09-02 - Transformaciones de Intensidad y Procesamiento de Histogramas]] — clase teórica
- [[2026-09-08 - Ecualización y Especificación de Histogramas]] — clase práctica de ecualización y matching
- [[Ecualización y Especificación de Histogramas - Python]] — código en OpenCV y scikit-image
- [[Inicio]] — mapa general de la materia

---

## 🏷️ Etiquetas

#visión-artificial #procesamiento-de-imágenes #histogramas #ecualización #contraste #histogram-matching #cdf

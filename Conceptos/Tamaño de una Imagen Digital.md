---
titulo: Tamaño de una Imagen Digital
materia: Visión Artificial
tipo: concepto
fecha: 2026-08-12
tags:
  - visión-artificial
  - procesamiento-de-imágenes
  - tamaño-de-imagen
aliases:
  - Fórmula b = N x M x k
  - Almacenamiento de imágenes
---

# Tamaño de una Imagen Digital

## 🎯 Definición

La cantidad de **bits** que ocupa una imagen digital depende de sus dimensiones (píxeles) y de los bits que usa cada píxel:

$$ b = N \times M \times k $$

| Símbolo | Significado | Unidad |
| :-----: | ----------- | :----: |
| $b$ | Tamaño total de la imagen | bits |
| $N$ | Número de **filas** (alto de la imagen) | píxeles |
| $M$ | Número de **columnas** (ancho de la imagen) | píxeles |
| $k$ | Bits por píxel (profundidad de bits) | bits |

## 🧠 Idea clave: la relación niveles ↔ bits

Cada píxel puede tomar **$L$ niveles** de intensidad, y $L$ se relaciona con los bits por píxel mediante:

$$ L = 2^k $$

- Un píxel de **1 bit** → $L = 2^1 = 2$ niveles (blanco o negro).
- Un píxel de **8 bits** → $L = 2^8 = 256$ niveles (grises del 0 al 255).

### Encontrar k cuando nos dan los niveles

Despejando $k$ de $L = 2^k$:

$$ k = \log_2(L) $$

| Niveles $L$ | $k = \log_2(L)$ | ¿Exacto? |
| :---------: | :-------------: | :------: |
| 256 | 8 | ✅ sí |
| 64 | 6 | ✅ sí |
| 8 | 3 | ✅ sí |
| 2 | 1 | ✅ sí |
| 20 | $\approx 4.32$ | ❌ no → se usan 5 bits |

> [!tip] L y k no siempre coinciden con enteros
> Cuando $L$ no es potencia de 2 (como 20 niveles), $k$ no es entero: $k = \log_2(20) = 4.32$. En la práctica se redondea hacia arriba y se usan **5 bits** (la nota del código dice "20 Niveles: 5bit").

## 🔍 Ejemplos

### 1. ¿Cuánto pesa una imagen 1024×1024 a 8 bits?

$$ b = 1024 \times 1024 \times 8 = 8\,388\,608 \text{ bits} \approx 1.05 \text{ MB} $$

### 2. Imagen 512×512 a 6 bits (64 niveles)

$$ b = 512 \times 512 \times 6 = 1\,572\,864 \text{ bits} \approx 0.19 \text{ MB} $$

### 3. Del código de clase: $L = 64$

$2^k = 64 \rightarrow k = 6$ → la imagen `img_64_L` usa **6 bits por píxel**.

## ✅ Conclusión

- El tamaño crece con los **píxeles** ($N \times M$) y con la **profundidad** ($k$).
- Reducir niveles (cuantizar) **reduce $k$** y por tanto el peso de la imagen, a costa de calidad.
- Para calcular $k$ a partir de los niveles siempre: $k = \log_2(L)$.

## 🔗 Relacionado

- [[Resolución de Imagen]] — la cuantización define $L$ (y por lo tanto $k$)
- [[Reducción de Niveles de Gris]] — cómo se reduce $L$ en la práctica
- [[Reducción de Niveles de Gris - Python]] — código que aplica niveles y muestra sus bits
- [[2026-08-12 - Fundamentos de Imágenes]] — clase donde se vio
- [[Inicio]] — mapa de contenidos

## 🏷️ Etiquetas

#visión-artificial #procesamiento-de-imágenes #tamaño-de-imagen

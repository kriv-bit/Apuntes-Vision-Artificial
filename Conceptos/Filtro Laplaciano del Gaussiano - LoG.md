---
titulo: "Filtro Laplaciano del Gaussiano - LoG"
materia: Visión Artificial
tipo: concepto
fecha: 2026-10-20
tags:
  - visión-artificial
  - procesamiento-de-imágenes
  - log
  - marr-hildreth
  - sombrero-mexicano
  - zero-crossings
  - multiescala
aliases:
  - LoG
  - Laplacian of Gaussian
  - Filtro de Marr-Hildreth
  - Sombrero Mexicano
  - Mexican Hat Filter
---

# Filtro Laplaciano del Gaussiano (LoG)

## 🎯 Definición y Origen

El **Filtro Laplaciano del Gaussiano (LoG)**, propuesto por **David Marr y Ellen Hildreth (1980)** en el MIT, nació como una solución fisiológica y computacional para superar la vulnerabilidad crítica del Laplaciano ante el ruido.

Combina dos conceptos fundamentales:
1. **Filtro Gaussiano $G_\sigma(x, y)$:** Actúa como filtro paso bajo para atenuar las altas frecuencias espaciales del ruido y definir la escala de inspección.
2. **Operador Laplaciano $\nabla^2$:** Actúa como operador de segunda derivada para detectar contornos nítidos a través de sus **cruces por cero (*zero-crossings*)**.

---

## 🧮 La Magia de la Conmutatividad

En un enfoque ingenuo, el procesamiento requeriría dos etapas computacionales:
$$ I_{filtrada} = \nabla^2 [f(x, y) * G_\sigma(x, y)] $$

Sin embargo, dado que la derivación y la convolución son **operadores lineales e invariantes en el desplazamiento**, son estrictamente conmutativos y asociativos:

$$ \nabla^2 [f(x, y) * G_\sigma(x, y)] = [\nabla^2 G_\sigma(x, y)] * f(x, y) $$

> [!tip] Ventaja Computacional Crítica
> En lugar de suavizar la imagen y luego calcular segundas derivadas sobre matrices discretas, se calcula **analíticamente la segunda derivada de la función Gaussiana continua**, obteniendo una única máscara convolucional precalculada: el kernel LoG.

---

## 📐 Formulación Matemática: El Sombrero Mexicano (*Mexican Hat*)

Partiendo de la función Gaussiana 2D circularmente simétrica:
$$ G_\sigma(x, y) = e^{-\frac{x^2 + y^2}{2\sigma^2}} $$

Aplicando el Laplaciano analítico continuo:
$$ \nabla^2 G_\sigma(x, y) = \frac{\partial^2 G_\sigma}{\partial x^2} + \frac{\partial^2 G_\sigma}{\partial y^2} $$

Obtenemos la ecuación canónica del **Sombrero Mexicano**:

$$ \text{LoG}(x, y) = -\frac{1}{\pi \sigma^4} \left( 1 - \frac{x^2 + y^2}{2\sigma^2} \right) e^{-\frac{x^2 + y^2}{2\sigma^2}} $$

```text
Perfil 1D de la función LoG:
                 /\
                /  \        <-- Lóbulo central positivo (máximo en r = 0)
    -----------+----+----------- 0 (Cruces por cero en r = ±√2 σ)
   /                    \
  +                      +   <-- Valles negativos laterales
```

* **Centro ($r = 0$):** Presenta un pico positivo pronunciado.
* **Cruces por Cero analíticos:** Ocurren exactamente a una distancia radial de:
  $$ r = \sqrt{x^2 + y^2} = \pm \sqrt{2}\sigma $$
* **Colas asintóticas:** Decaen suavemente a cero para $r \to \infty$.

---

## ⚖️ El Rol del Parámetro $\sigma$ (Escala de Detección)

El parámetro $\sigma$ (desviación estándar) controla la **escala de los bordes** que se desean detectar:

| Valor de $\sigma$ | Ancho del Kernel | Comportamiento en la Imagen |
| :---: | :---: | :--- |
| **$\sigma$ pequeño ($0.8 - 1.2$)** | Ventana pequeña ($5\times 5$ a $9\times 9$) | Detecta detalles finos, texturas y bordes de alta resolución. Menor inmunidad a ruido residual. |
| **$\sigma$ grande ($2.5 - 4.0$)** | Ventana amplia ($17\times 17$ a $25\times 25$) | Suprime ruido severo (ej. speckle en ecografías o grano térmico). Detecta solo los contornos macroscópicos y estructuras globales. |

> [!important] Regla del Tamaño del Kernel Discreto
> Para muestrear adecuadamente la función LoG continua sin truncar sus valles negativos, el tamaño del kernel debe ser impar y dimensionado como mínimo a:
> $$ K_{size} \approx 6\sigma + 1 \quad \text{(redondeado al siguiente impar)} $$

---

## 🔗 Relacionado

- [[Operador Laplaciano y Cruces por Cero]]
- [[Derivadas Espaciales en Imágenes]]
- [[Filtros de Suavizado Espacial]]
- [[Operador Laplaciano y Filtro LoG - Python]]

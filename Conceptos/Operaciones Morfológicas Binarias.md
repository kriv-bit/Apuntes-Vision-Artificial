---
titulo: "Operaciones Morfológicas Binarias"
materia: Visión Artificial
tipo: concepto
fecha: 2026-11-03
unidad: 5
tags:
  - visión-artificial
  - procesamiento-de-imágenes
  - morfología-matemática
  - imágenes-binarias
  - erosión
  - dilatación
  - apertura
  - cierre
  - elemento-estructurante
aliases:
  - Morfología Matemática Binaria
  - Erosión y Dilatación
  - Apertura y Cierre Morfológico
  - Dualidad Morfológica
---

# Operaciones Morfológicas Binarias

## 🎯 Definición

La **Morfología Matemática** es una teoría no lineal de procesamiento de imágenes basada en la **teoría de conjuntos**, la **geometría integral** y el **retículo de Boole**. Fue desarrollada originalmente a mediados de la década de 1960 por Georges Matheron y Jean Serra en la Escuela de Minas de París.

En imágenes binarias, una imagen $A$ se modela como un subconjunto del espacio bidimensional discreto $\mathbb{Z}^2$, donde los elementos del conjunto son las coordenadas espaciales $(x, y)$ de los píxeles de primer plano (valor 1):

$$A = \{ (x, y) \in \mathbb{Z}^2 \mid I(x, y) = 1 \}$$

El objetivo de las operaciones morfológicas es extraer componentes de la imagen que son útiles en la representación y descripción de la **forma** de regiones, tales como límites, esqueletos, convexidad o conectividad, mediante la interacción de $A$ con un conjunto sonda denominado **Elemento Estructurante** ($B$).

---

## 🔬 1. Elemento Estructurante (Structuring Element - SE)

El elemento estructurante $B \subset \mathbb{Z}^2$ es una pequeña forma geométrica predefinida con un punto origen designado (generalmente el centro de gravedad o coordenada $(0,0)$).

### Formas comunes de $B$:
- **Cruz ($N_4$)**: Vecindario de 4 conectividad.
- **Rectángulo / Cuadrado ($N_8$)**: Ventana $3\times 3$, $5\times 5$, etc.
- **Elipse / Disco**: Isotropía rotacional aproximada.
- **Línea**: Detección o preservación de orientaciones direccionales.

---

## 📐 2. Operaciones Fundamentales

### 2.1. Erosión ($A \ominus B$)
La erosión determina qué posiciones del elemento estructurante $B$ quedan **completamente contenidas** en el objeto $A$:

$$A \ominus B = \{ z \in \mathbb{Z}^2 \mid (B)_z \subseteq A \}$$

donde $(B)_z = \{ b + z \mid b \in B \}$ representa la traslación del conjunto $B$ por el vector $z$.

Equivalencia como intersección de traslaciones:
$$A \ominus B = \bigcap_{b \in B} (A)_{-b}$$

#### Efectos:
- Contracción o adelgazamiento del objeto.
- Eliminación de estructuras y protuberancias menores que la forma de $B$.
- Ruptura de puentes estrechos (*istmos*) entre componentes.

---

### 2.2. Dilatación ($A \oplus B$)
La dilatación busca todas las posiciones donde la reflexión de $B$ tiene **al menos un punto de intersección** con $A$:

$$A \oplus B = \{ z \in \mathbb{Z}^2 \mid (\hat{B})_z \cap A \neq \emptyset \}$$

donde $\hat{B} = \{ -b \mid b \in B \}$ es la reflexión de $B$ respecto al origen.

Equivalencia como unión de traslaciones:
$$A \oplus B = \bigcup_{b \in B} (A)_b$$

#### Efectos:
- Expansión o engrosamiento del objeto.
- Relleno de agujeros interiores y porosidades.
- Unión o fusión de objetos cercanos desconectados.

---

## 🔄 3. Principio de Dualidad

Erosionar un objeto es idéntico a dilatar el fondo, y viceversa. Matemáticamente:

$$(A \ominus B)^c = A^c \oplus \hat{B}$$

$$(A \oplus B)^c = A^c \ominus \hat{B}$$

donde $A^c = \{ w \in \mathbb{Z}^2 \mid w \notin A \}$ representa el complemento de $A$ (el fondo).

> [!important] Relevancia Algorítmica
> Gracias al principio de dualidad, cualquier procesador de imágenes solo necesita implementar el hardware o la rutina de **erosión** (o de dilatación); la otra operación se obtiene simplemente complementando antes y después.

---

## 🧼 4. Operaciones Morfológicas Compuestas

### 4.1. Apertura Morfológica ($A \circ B$)
La apertura es la **erosión seguida de dilatación** con el mismo elemento estructurante $B$:

$$A \circ B = (A \ominus B) \oplus B$$

Geométricamente, $A \circ B$ es la unión de todas las traslaciones de $B$ que caben completamente dentro de $A$:
$$A \circ B = \bigcup \{ (B)_z \mid (B)_z \subseteq A \}$$

#### Propiedades algebraicas:
1. **Anti-extensiva**: $A \circ B \subseteq A$ (no crea nuevos píxeles).
2. **Idempotente**: $(A \circ B) \circ B = A \circ B$ (un segundo filtrado no cambia el resultado).
3. **Creciente (Monótona)**: Si $A_1 \subseteq A_2 \implies (A_1 \circ B) \subseteq (A_2 \circ B)$.

---

### 4.2. Cierre Morfológico ($A \bullet B$)
El cierre es la operación dual de la apertura: consiste en una **dilatación seguida de erosión** con el mismo elemento estructurante $B$:

$$A \bullet B = (A \oplus B) \ominus B$$

#### Propiedades algebraicas:
1. **Extensiva**: $A \subseteq A \bullet B$ (el objeto crece para sellar huecos).
2. **Idempotente**: $(A \bullet B) \bullet B = A \bullet B$.
3. **Creciente (Monótona)**: Si $A_1 \subseteq A_2 \implies (A_1 \bullet B) \subseteq (A_2 \bullet B)$.
4. **Dualidad con la Apertura**: $(A \bullet B)^c = A^c \circ \hat{B}$.

---

## 📐 5. Extracción Morfológica de Bordes

Restando la versión erosionada de la imagen original, se obtiene la frontera perimetral del objeto:

$$\text{Original} - \text{Erosión} = \text{Borde Interno}$$
$$\beta(A) = A \setminus (A \ominus B) = A - (A \ominus B)$$

- **Borde externo**: $\beta_{\text{ext}}(A) = (A \oplus B) - A$.
- **Gradiente morfológico**: $G(A) = (A \oplus B) - (A \ominus B)$.

---

## 📊 Comparativa Rápida de Propiedades

| Propiedad | Erosión ($A \ominus B$) | Dilatación ($A \oplus B$) | Apertura ($A \circ B$) | Cierre ($A \bullet B$) |
| :--- | :--- | :--- | :--- | :--- |
| **Extensividad** | Anti-extensiva ($A \ominus B \subseteq A$) | Extensiva ($A \subseteq A \oplus B$) | Anti-extensiva ($A \circ B \subseteq A$) | Extensiva ($A \subseteq A \bullet B$) |
| **Idempotencia** | No (continúa adelgazando) | No (continúa engrosando) | **Sí** ($X \circ B = X$) | **Sí** ($X \bullet B = X$) |
| **Preserva Tamaño Global** | No (reduce masa) | No (aumenta masa) | **Sí** (remueve solo picos/islas) | **Sí** (sella solo grietas/huecos) |
| **Filtro de Ruido** | Ruido exterior (pero encoge) | Ruido interior (pero agranda) | Ruido de fondo (islas blancas) | Ruido en objetos (agujeros negros) |

---

## 🔗 Notas Relacionadas
- [[2026-11-03 - Morfología Matemática - Binaria y Escala de Grises]] — Clase teórica y contextualización de la sesión.
- [[Transformación Hit-or-Miss]] — Detección exacta de patrones, esquinas y minucias de huellas dactilares.
- [[Adelgazamiento, Espesamiento y Esqueletización]] — Preservación topológica y reducción a esqueletos.
- [[Morfología en Escala de Grises]] — Superficies topográficas 3D, máximos y mínimos locales.
- [[Morfología Matemática y Hit-or-Miss - Python]] — Implementación práctica con `cv2.erode`, `cv2.dilate` y `cv2.morphologyEx`.
- [[Operaciones Lógicas y Álgebra Booleana en Imágenes]] — Conjuntos y máscaras booleanas.
- [[Inicio]] — Hub principal de la asignatura.

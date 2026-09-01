---
titulo: Inicio
tipo: home
materia: Visión Artificial
tags:
  - visión-artificial
---

# 🎓 Visión Artificial

> [!info] Materia electiva
> Bóveda de apuntes de la electiva **Visión Artificial**.
> Organizada por **clases** (con fecha), **conceptos** (notas atómicas) y **código** (scripts explicados paso a paso), todo enlazado bidireccionalmente con wikilinks.

## 📅 Clases (por fecha)

> [!tip] Convención
> Cada clase se guarda en `Clases/` con el formato `YYYY-MM-DD - Título` para que se ordenen cronológicamente de forma automática.

- [[2026-08-12 - Fundamentos de Imágenes]] — fundamentos, niveles de gris, resolución y Python (Colab)
- [[2026-08-18 - Interpolación y Redimensionamiento]] — escalado $3\times$, interpolación (Nearest, Linear, Cubic, Exact) y Zoom en OpenCV
- [[2026-08-19 - Relaciones entre Píxeles y Métricas de Distancia]] — vecindades ($N_4, N_D, N_8$), $m$-adyacencia, caminos y distancias ($D_e, D_4, D_8$)
- [[2026-09-01 - Operaciones Aritméticas y Lógicas entre Imágenes]] — suma/promedio, resta, multiplicación, división, desbordamiento, saturación y normalización

## 🧠 Conceptos

- [[Reducción de Niveles de Gris]] — cuantización de 256 a 8 niveles
- [[Tamaño de una Imagen Digital]] — $b = N \times M \times k$ y $k = \log_2(L)$
- [[Resolución de Imagen]] — muestreo (espacial) vs cuantización (intensidad)
- [[Interpolación]] — vecino más cercano, bilineal, bicúbica y variantes exactas
- [[Vecindad y Adyacencia de Píxeles]] — $N_4, N_D, N_8$, conjunto $V$, 4/8-adyacencia y $m$-adyacencia
- [[Métricas de Distancia en Imágenes]] — formulación y cálculo de $D_e, D_4, D_8$
- [[Operaciones Aritméticas entre Imágenes]] — suma, resta, multiplicación, división y aplicaciones
- [[Desbordamiento y Normalización de Imágenes]] — overflow modular, saturación y reescalado Min-Max
- [[Operaciones Lógicas y Álgebra Booleana en Imágenes]] — operadores binarios AND, OR, NOT, XOR y enmascaramiento

## 💻 Código

- [[Reducción de Niveles de Gris - Python]] — cuantización a 64, 20, 8 y 2 niveles con NumPy y Pillow
- [[Interpolación y Redimensionamiento - Python]] — redimensionamiento con `cv2.resize`, métodos clásicos y variantes `_EXACT` con inspección por zoom
- [[Métricas de Distancia y Conectividad - Python]] — funciones de cálculo de $D_e, D_4, D_8$ y mapas de isolíneas
- [[Operaciones Aritméticas y Normalización - Python]] — operaciones punto a punto, overflow, saturación y mezcla ponderada en OpenCV/scikit-image

## 🗺️ Estructura de la bóveda

```mermaid
graph TD
    Inicio[🏠 Inicio] --> C1[Clase 2026-08-12 Fundamentos]
    Inicio --> C2[Clase 2026-08-18 Interpolación]
    Inicio --> C3[Clase 2026-08-19 Relaciones y Distancias]
    Inicio --> C4[Clase 2026-09-01 Operaciones y Normalización]
    
    C1 --> N1[Reducción de Niveles de Gris]
    C1 --> N2[Tamaño de Imagen Digital]
    C1 --> N3[Resolución de Imagen]
    C1 --> N4[Interpolación]
    C1 --> P1[Python - Cuantización]
    
    C2 --> N4
    C2 --> N3
    C2 --> P2[Python - Interpolación OpenCV]
    
    C3 --> N5[Vecindad y Adyacencia]
    C3 --> N6[Métricas de Distancia]
    C3 --> P3[Python - Distancias De D4 D8]
    
    C4 --> N7[Operaciones Aritméticas]
    C4 --> N8[Desbordamiento y Normalización]
    C4 --> N9[Operaciones Lógicas]
    C4 --> P4[Python - Operaciones y Normalización]
    
    N7 --> N8
    N8 --> P4
```

## 🛠️ Plantillas

- [[Plantilla de Clase]] — apuntes por sesión (con fecha)
- [[Plantilla de Concepto]] — notas atómicas por tema
- [[Plantilla de Código]] — scripts explicados

## 📌 ¿Cómo uso esta bóveda?

1. **Nueva clase** → usa [[Plantilla de Clase]] y nombra la nota `YYYY-MM-DD - Tema`.
2. **Concepto nuevo** → nota atómica en `Conceptos/` con la [[Plantilla de Concepto]].
3. **Código nuevo** → nota en `Código/` con la [[Plantilla de Código]].
4. Enlaza todo con `[[wikilinks]]`: clase ↔ concepto ↔ código.

## 📊 Vista dinámica (plugin Dataview)

> [!note] Opcional
> Si tienes instalado el plugin comunitario **Dataview**, estas tablas listan automáticamente tus clases y scripts:

```dataview
TABLE fecha, semana
FROM "Clases"
SORT fecha DESC
```

```dataview
TABLE fecha, clase as "Clase asociada"
FROM "Código"
SORT fecha DESC
```

## 🏷️ Etiquetas principales

- `#visión-artificial` · `#clase` · `#python` · `#opencv` · `#interpolación` · `#vecindad` · `#adyacencia` · `#distancias` · `#operaciones-aritméticas` · `#normalización`

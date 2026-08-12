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
> Organizada por **clases** (con fecha), **conceptos** y **código**, todo enlazado entre sí con wikilinks.

## 📅 Clases (por fecha)

> [!tip] Convención
> Cada clase se guarda en `Clases/` con el formato `YYYY-MM-DD - Título` para que se ordenen solas.

- [[2026-08-12 - Fundamentos de Imágenes]] — fundamentos, niveles de gris y Python

## 🧠 Conceptos

- [[Reducción de Niveles de Gris]] — cuantización de 256 a 8 niveles

## 💻 Código

- [[Reducción de Niveles de Gris - Python]] — implementación en NumPy/Matplotlib (Colab)

## 🗺️ Estructura de la bóveda

```mermaid
graph LR
    Inicio[🏠 Inicio] --> Clases
    Inicio --> Conceptos
    Inicio --> Codigo[Código]
    Clases --> C1[Clase 2026-08-12]
    C1 --> N1[Reducción de Niveles de Gris]
    C1 --> P1[Python]
    N1 --> P1
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
> Instala el plugin comunitario **Dataview** y esta consulta listará las clases automáticamente, ordenadas por fecha:

```dataview
TABLE fecha, semana
FROM "Clases"
SORT fecha DESC
```

## 🏷️ Etiquetas principales

- `#visión-artificial` · `#clase` · `#python` · `#niveles-de-gris`

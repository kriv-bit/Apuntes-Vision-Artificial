# 🎓 Bóveda de Apuntes — Visión Artificial

Bóveda de [Obsidian](https://obsidian.md/) estructurada bajo principios de gestión de conocimiento personal (**PKM** / *Linking Your Thinking*), notas atómicas y enlaces bidireccionales.

---

## 🗂️ Estructura del Vault

```text
Apuntes-Vision-Artificial/
├── 🏠 Inicio.md                  # Hub central / Map of Content (MOC)
├── 📅 Clases/                    # Bitácoras de sesión (YYYY-MM-DD - Título.md)
│   ├── 2026-08-12 - Fundamentos de Imágenes.md
│   └── 2026-08-18 - Interpolación y Redimensionamiento.md
├── 🧠 Conceptos/                 # Notas atómicas (un concepto por archivo)
│   ├── Interpolación.md
│   ├── Reducción de Niveles de Gris.md
│   ├── Resolución de Imagen.md
│   └── Tamaño de una Imagen Digital.md
├── 💻 Código/                    # Scripts explicados paso a paso
│   ├── Reducción de Niveles de Gris - Python.md
│   └── Interpolación y Redimensionamiento - Python.md
├── 🖼️ Adjuntos/                  # Diagramas, capturas y recursos gráficos
│   ├── comparacion-interpolacion-zoom.png
│   └── comparacion-interpolacion-exacta.png
└── 🛠️ Templates/                 # Plantillas estandarizadas para notas
    ├── Plantilla de Clase.md
    ├── Plantilla de Concepto.md
    └── Plantilla de Código.md
```

---

## ⚡ Buenas prácticas implementadas (Obsidian Pro Workflow)

1. **Map of Content (MOC)**: [`Inicio.md`](file:///C:/Users/Udenar/Documents/Obsidian%20Vault/Apuntes-Vision-Artificial/Inicio.md) actúa como panel de control que centraliza y navega todo el conocimiento.
2. **Notas Atómicas**: Cada concepto teórico vive en su propia nota independiente en `Conceptos/`, facilitando su reutilización y conexión en el grafo.
3. **Wikilinks Bidireccionales**: Relación directa entre teoría $\leftrightarrow$ sesiones de clase $\leftrightarrow$ código fuente.
4. **Frontmatter YAML enriquecido**: Metadatos estándar (`materia`, `tipo`, `fecha`, `tags`, `aliases`) listos para consultas con **Dataview**.
5. **Callouts y MathJax**: Destacados visuales (`> [!info]`, `> [!tip]`, etc.) y renderizado de fórmulas LaTeX ($$...$$).
6. **Gestión de Recursos**: Imágenes y recursos multimedia organizados en `Adjuntos/` con nombres semánticos.

---

## 🔌 Plugins recomendados para esta bóveda

- **Dataview**: Para tablas dinámicas automáticas de clases y código.
- **Paste Image Rename**: Para nombrar automáticamente las imágenes pegadas en la carpeta `Adjuntos/`.
- **Omnisearch**: Búsqueda semántica rápida en notas, PDFs y código.

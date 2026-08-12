# AGENTS.md — Bóveda de Obsidian: Visión Artificial

## Contexto del proyecto

Esta es una **bóveda de Obsidian** con los apuntes de la materia electiva **Visión Artificial**. Todo el contenido es Markdown plano. Los agentes que trabajen aquí deben **respetar las convenciones de Obsidian** para que los wikilinks, el grafo y los plugins sigan funcionando correctamente.

- **Idioma del contenido:** español (los términos técnicos pueden conservarse en inglés).
- **Hoy:** usa siempre la fecha actual del sistema en formato ISO.

## Estructura de carpetas (obligatoria)

| Carpeta | Contenido |
| ------- | --------- |
| `Clases/` | Una nota por sesión, nombrada `YYYY-MM-DD - Título.md` |
| `Conceptos/` | Notas atómicas: **un concepto por nota** |
| `Código/` | Scripts explicados paso a paso |
| `Templates/` | Plantillas para crear notas nuevas |
| `Inicio.md` | Hub central (MOC). Toda nota debe enlazarse desde aquí |

**Regla:** nunca crees carpetas nuevas para notas sueltas; usa las existentes según el tipo de nota.

## Convenciones de Obsidian

### 1. Frontmatter YAML (obligatorio en toda nota)

```yaml
---
titulo: "..."
materia: Visión Artificial
tipo: home | clase | concepto | código | plantilla
fecha: YYYY-MM-DD
tags:
  - visión-artificial
---
```

- `fecha` **siempre** en formato ISO `YYYY-MM-DD` — nunca `DD/MM/YYYY`.
- `tipo` determina la carpeta donde vive la nota.
- Las clases llevan además `semana:` y un enlace `clase: "[[...]]"` cuando aplique.
- `aliases:` solo cuando el tema tenga nombres alternativos.

### 2. Nombres de archivo

- Clases: `YYYY-MM-DD - Título.md` (el prefijo ISO las ordena cronológicamente).
- Conceptos y código: `Título Corto.md`, sin fechas ni prefijos numéricos.
- Evita caracteres problemáticos: `: / * ? " < > |`.

### 3. Enlaces

- **Siempre wikilinks** `[[Nombre de la Nota]]` — nunca rutas relativas ni absolutas.
- Toda nota nueva se enlaza con `[[Inicio]]`.
- Relación clase ↔ concepto ↔ código: enlázalas **entre sí** (bidireccional).

### 4. Etiquetas (tags)

- Minúsculas y sin espacios: usa `-` (`#niveles-de-gris`).
- Jerárquicas cuando aplique: `#visión-artificial/clase`.
- Reutiliza las etiquetas que ya existen en la bóveda; no inventes sinónimos.

### 5. Fórmulas

- LaTeX/MathJax de Obsidian: `$...$` en línea y `$$...$$` en bloque.
- Una fórmula larga por bloque, con saltos de línea legibles.

### 6. Diagramas Mermaid

- Usa bloques `mermaid`.
- **Sin HTML inline en los nodos** (no usar `<br/>`).
- **Sin colores hardcodeados ni `classDef`**: Obsidian los tiñe automáticamente con el tema del usuario.
- Prefiere diagramas **más altos que anchos** (la vista renderizada puede ser angosta).

### 7. Callouts

- Úsalos para destacar: `> [!info]`, `> [!tip]`, `> [!warning]`, `> [!important]`, `> [!note]`.
- Texto breve después del tipo de callout.

### 8. Estilo de escritura

- Notas atómicas: **un concepto por nota**.
- **No dupliques contenido**: referencia con wikilinks en lugar de copiar.
- Secciones con emojis de cabecera, como en las notas existentes:
  `🎯 Objetivo`, `🧠 Idea clave`, `🧮 Desarrollo`, `🔍 Ejemplos`, `✅ Conclusión`, `🔗 Relacionado`.
- Código: incluye siempre requisitos, explicación paso a paso, resultado esperado y errores comunes.
- No agregues comentarios que solo repiten el código.

### 9. Plantillas

- Usa siempre las plantillas de `Templates/` para notas nuevas:
  - `Plantilla de Clase.md`
  - `Plantilla de Concepto.md`
  - `Plantilla de Código.md`
- No inventes estructuras nuevas si existe una plantilla para ese tipo.

## Reglas de edición

- No rompas la configuración de `.obsidian/` (`app.json`, `core-plugins.json`, etc.).
- `workspace.json` y `.trash/` están en `.gitignore`: no los modifiques ni los subas.
- Antes de renombrar o mover una nota, actualiza **todos** sus wikilinks.
- Si la tarea implica reorganizar la bóveda (mover carpetas, renombrar, reestructurar), **propón primero el plan al usuario** antes de editar.
- Mantén los cambios mínimos y enfocados; no toques notas no relacionadas.

## Git

- Mensajes de commit en **español**, en imperativo, sujeto corto (≤ 50 caracteres) y cuerpo a 72 columnas solo si aporta información útil.
- No hagas commit ni push salvo que el usuario lo pida.

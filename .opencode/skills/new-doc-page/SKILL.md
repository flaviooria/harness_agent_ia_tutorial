---
name: new-doc-page
description: Crear una nueva página de documentación en Zensical
agents: [docwriter]
---

# Skill: new-doc-page

Cómo crear una página de documentación nueva en este proyecto.

## 1. Ubicación

Todas las páginas van en `docs/`. Usa subdirectorios si el contenido lo requiere.

## 2. Nombre del archivo

- `snake_case.md` para el archivo
- Directorios en `kebab-case`

## 3. Frontmatter YAML

Toda página necesita frontmatter YAML. Campos obligatorios:

```yaml
---
title: "Título de la página"
icon: lucide/icono
---
```

Para elegir un icono Lucide, consulta https://lucide.dev/icons.
Usa el prefijo `lucide/` seguido del nombre del icono en kebab-case.

## 4. Estructura de la página

- Usa `#` para el título principal (H1) — normalmente coincide con `title`
- Usa `##` para secciones (H2), `###` para subsecciones (H3)
- Incluye ejemplos de código cuando sea relevante
- Usa `!!! note "Título"` para admonitions (note, tip, warning)
- Usa `<div class="grid cards">` para tarjetas (grid cards)
- Usa tablas con `|` para datos estructurados

## 5. Enlaces entre páginas

Usa rutas relativas desde `docs/`:

```markdown
[texto del enlace](pagina.md)
[texto del enlace](subdirectorio/pagina.md)
```

## 6. Verificación

```sh
uv run zensical build --clean
```

No debe mostrar errores ni warnings.

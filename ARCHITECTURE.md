# ARCHITECTURE.md — Harness Agent Guide

## Tipo de proyecto

Guía educativa para construir Harness Agents desde cero.
**100% documentación** — no hay código fuente de aplicación.

Stack: Python 3.12, Zensical (generador de documentación), uv.

## Estructura del repositorio

```
docs/              → páginas Zensical en Markdown con frontmatter YAML
  index.md         → introducción, mapa del ecosistema
  harness.md       → los 5 roles del harness (contenido educativo)
  base_docs.md     → AGENTS.md y ARCHITECTURE.md de referencia
  markdown.md      → referencia rápida de Markdown
  stylesheets/     → CSS personalizado (paleta "cozy")
  references.md    → referencias de flujo de trabajo, mcp y pipeline

.opencode/         → configuración de agentes OpenCode
  agents/          → definiciones de roles del harness
  commands/        → slash commands (/harness-run, /harness-proposal)
  skills/          → skills para tareas concretas

openspec/          → template de referencia (vacío, usado en ejemplos educativos)
```

## Pipeline

```sh
uv run zensical build --clean   # genera site/ (estático)
uv run zensical serve           # servidor de desarrollo
```

## Roles activos del harness

```
orchestrator → docwriter → reviewer
```

Los 5 roles completos (incluyendo backend y tester) se enseñan como contenido educativo en `docs/harness.md`.

## Skills disponibles

- `new-doc-page` → creación de páginas de documentación
- `doc-review`   → revisión de calidad de documentación

# AGENTS.md — Harness Agent Guide

> Este archivo es leído automáticamente por OpenCode, Claude Code, Cursor
> y la mayoría de agentes antes de cualquier tarea. No es el README.

## Resumen del proyecto

Guía educativa para construir Harness Agents desde cero. Stack: Python 3.12, Zensical (generador de documentación), uv. Todo el contenido está en Markdown — no hay código fuente de aplicación.

## Antes de empezar CUALQUIER tarea

1. Lee ARCHITECTURE.md (si existe) para entender la estructura del proyecto.
2. Comprueba que entiendes los roles del harness en docs/harness.md.
3. Si la tarea implica contenido nuevo, define primero el alcance.

## Comandos de entorno

- Instalar dependencias:   uv sync
- Build docs local:        uv run zensical build --clean
- Servir docs local:       uv run zensical serve
- Build docs + release:    zensical build --clean && cp -r site/* .

## Estilo de contenido

- Markdown con frontmatter YAML. Todas las páginas en docs/.
- Archivos en snake_case.md. Directorios en kebab-case.
- Iconos Lucide para cada página (icon: lucide/...).
- Español como idioma del sitio (configurado en zensical.toml).
- CSS personalizado en docs/stylesheets/extra.css — paleta "cozy" con dark mode.

## Flujo del harness

orchestrator → docwriter → reviewer
En este proyecto (100% documentación) solo se usan orchestrator, docwriter y reviewer.
Los 5 roles completos se enseñan como contenido educativo en docs/harness.md.

## Skills

Antes de implementar, lee la skill relevante en `.opencode/skills/`.
- `new-doc-page` para crear páginas de documentación
- `doc-review` para revisar calidad

## Qué NO hacer

- No mezclar contenido de documentación con código fuente.
- No añadir dependencias Python de producción (dependencies = [] debe seguir vacío).
- No modificar zensical.toml sin entender las features de Zensical.
- No commitear el directorio site/ (generado) — está en .gitignore.

## PRs

Formato título: [agent-harness] descripción corta
Antes de abrir PR: uv run zensical build --clean para verificar que buildea sin errores.

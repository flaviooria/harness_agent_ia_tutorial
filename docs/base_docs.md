---
title: Documentos Base
icon: lucide/file-text
---

## AGENTS.md — El contrato con la IA

!!! note "🏠 Analogía"
    AGENTS.md son las normas pegadas en la nevera cuando llegas a un Airbnb: "el Wi-Fi es X, el cubo de basura está aquí, no hagas ruido después de las 22h". El agente las lee antes de hacer cualquier cosa, igual que tú las lees antes de instalarte.


AGENTS.md es el único archivo que todo agente lee siempre, sin excepción, antes de tocar nada. No es el README del proyecto (ese es para humanos). Es el contrato entre tú y el agente sobre cómo se trabaja en este proyecto.<sup>[[3]](references.md#ref3)</sup>

!!! tip "✅ Estándar oficial"
    El formato AGENTS.md está mantenido por la Agentic AI Foundation bajo la Linux Foundation. Es compatible de forma nativa con OpenCode, Claude Code, Codex, Jules (Google), Cursor, Devin, GitHub Copilot Agent y más de 60.000 proyectos open source. <sup>[[3]](references.md#ref3)</sup>

**Estructura recomendada**

> AGENTS.md (El contrato con la IA), ejemplo de un proyecto de lista de tareas:

```markdown
# AGENTS.md — Todo Harness

> Este archivo es leído automáticamente por OpenCode, Claude Code, Cursor
> y la mayoría de agentes antes de cualquier tarea. No es el README.

## Resumen del proyecto
API REST de una lista de tareas. Stack: Python 3.12, FastAPI, SQLAlchemy
async 2.0, PostgreSQL, Docker, uv workspaces.

## Antes de empezar CUALQUIER tarea
1. Lee ARCHITECTURE.md para entender la estructura del sistema.
2. Si la tarea implica funcionalidad nueva → el spec debe existir en
   openspec/changes/ antes de escribir código.
   Créalo con /harness-proposal si no existe.
3. Identifica el agente responsable leyendo agents/orchestrator.md.
4. Lee la skill relevante en .opencode/skills/ antes de implementar.

## Comandos de entorno
- Instalar dependencias:   uv sync
- Levantar BD + API:       docker compose up -d
- Correr API en local:     uv run uvicorn app.main:app --reload
- Correr tests:            uv run pytest
- Test específico:         uv run pytest -k "nombre_del_test"
- Nueva migración:         uv run alembic revision --autogenerate -m "msg"
- Aplicar migraciones:     uv run alembic upgrade head
- Linting:                 uv run ruff check .

## Estilo de código
- Python 3.12+ con type hints en todas las funciones. Prohibido Any.
- SQLAlchemy 2.0 async: usar AsyncSession y select(). Nunca síncrono.
- Pydantic v2 para todos los schemas. Nunca devolver modelos de ORM.
- Nombres de tabla: snake_case plural (todos, no Todo ni todo).
- Cada recurso en su propio router: app/routers/{recurso}.py.

## Flujo del harness
orchestrator → backend → tester → reviewer → docwriter
El orquestador puede saltarse pasos según el tipo de tarea.
Ver agents/orchestrator.md para el protocolo completo.

## MCPs disponibles
- postgres:  consultar/inspeccionar BD directamente (debug, verificación)
- github:   crear PRs, leer issues, añadir comentarios

## Qué NO hacer
- No commitear directamente a main. Siempre rama + PR.
- No modificar openspec/specs/ manualmente.
- No poner lógica de negocio en routers ni SQL en services.
- No usar print() en código final; usar logging.

## PRs
Formato título: [todo-harness] descripción corta
Antes de abrir PR: uv run pytest y uv run ruff check . en verde.
Enlazar en la descripción al change de openspec correspondiente.
```

- **Reglas de buena escritura**
    - **Sé específico sobre los comandos**: escribe exactamente `uv run pytest`, no "corre los tests".
    - **Prohíbe explícitamente lo que no quieres**: la sección "Qué NO hacer" es tan importante como las instrucciones positivas.
    - **Referencia otros documentos con rutas relativas**: el agente las puede seguir.
    - **Mantén AGENTS.md lean**: el detalle va en skills y en agents/. AGENTS.md es el índice, no la enciclopedia.

## ARCHITECTURE.md — El mapa del sistema   

Mientras AGENTS.md *define cómo trabajar*, ARCHITECTURE.md explica *cómo está construido el sistema*. Es el documento que el orquestador lee para entender qué agente necesita cada tarea.

> ARCHITECTURE.md (El mapa del sistema), ejemplo de un proyecto de lista de tareas:

```markdown
# ARCHITECTURE.md — Todo Harness

# Capas del sistema

HTTP (routers)     app/routers/        ← solo valida input, enruta, devuelve schema
Business (services) app/services/       ← lógica de negocio, sin conocimiento HTTP
Data (repositories) app/repositories/   ← queries SQLAlchemy async, nada de negocio
Schemas             app/schemas/        ← Pydantic v2 I/O, nunca modelos ORM
Models              app/models/         ← SQLAlchemy declarative models
Migrations          alembic/            ← versionado de BD

## Regla de capas (CRÍTICA)
Router → Service → Repository. En ese orden, nunca al revés.
Un router NO llama a un repository directamente.
Un service NO importa nada de FastAPI (ni Request, ni HTTPException directo).

## Árbol de módulos
app/
├── main.py              ← FastAPI app, exception handlers globales, routers registrados
├── dependencies.py      ← inyección de dependencias (get_db, get_service...)
├── routers/             ← un archivo por recurso
├── services/            ← un archivo por dominio
├── repositories/        ← un archivo por modelo
├── schemas/             ← {recurso}.py con Create / Read / Update
└── models/              ← {recurso}.py con el modelo SQLAlchemy

## Agentes y responsabilidades
backend.md    → código nuevo, endpoints, models, services, repositories
tester.md     → tests unitarios e integración, fixtures, cobertura
reviewer.md   → calidad, adherencia a capas, seguridad, type hints
docwriter.md  → docstrings, README, descripciones OpenAPI

## Decisiones de diseño relevantes
- AsyncSession: toda la capa de datos usa async/await. Nunca sync con greenlets.
- Alembic con autogenerate: los modelos son la fuente de verdad del esquema.
- Pydantic v2: validators con @field_validator, no @validator.
- Todas las excepciones de dominio heredan de DomainError y son
  capturadas por un handler global en main.py (nunca HTTPException en services).

## MCPs disponibles y cuándo usarlos
postgres: el reviewer usa este MCP para verificar que las migraciones
          generaron el esquema correcto. Comando ejemplo: describe table todos.
github:   el docwriter y el orchestrator usan este MCP para crear PRs
          y enlazar el change de openspec.
```
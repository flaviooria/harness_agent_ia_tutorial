# agent-harness

Guía educativa para construir **Harness Agents** desde cero. Todo el contenido está en Markdown — no hay código fuente de aplicación.

**Stack**: Python 3.12 · Zensical (generador de documentación estática) · uv

---

## ¿Qué vas a aprender?

- Qué es un agente IA y qué es un harness
- Los 5 roles del harness: orchestrator, backend, tester, reviewer, docwriter
- Cómo crear agents, skills y commands en OpenCode
- El estándar OpenSpec para spec-driven development
- Cómo integrar MCP (Model Context Protocol)
- Cómo añadir memoria persistente con Engram

---

## Requisitos

- Python 3.12+
- uv (gestor de paquetes Python)

```bash
# Instalar uv si no lo tienes
curl -LsSf https://astral.sh/uv/install.sh | sh
```

---

## Primeros pasos

```bash
# 1. Clonar el repositorio
git clone https://github.com/tu-usuario/agent-harness.git
cd agent-harness

# 2. Instalar dependencias (Zensical)
uv sync

# 3. Build de la documentación
uv run zensical build --clean

# 4. Servir en local (http://127.0.0.1:8000)
uv run zensical serve
```

---

## Estructura del proyecto

```
agent-harness/
├── AGENTS.md              ← Reglas para agentes IA (OpenCode, Claude Code...)
├── ARCHITECTURE.md        ← Mapa del sistema
├── pyproject.toml         ← Proyecto Python (dependencies = [])
├── zensical.toml          ← Configuración de Zensical
│
├── docs/                  ← Documentación en Markdown
│   ├── index.md           ← Introducción al harness
│   ├── harness.md         ← Los 5 roles, skills, commands, specs, MCP
│   ├── base_docs.md       ← Referencia de AGENTS.md y ARCHITECTURE.md
│   ├── engram.md          ← Memoria persistente con Engram
│   ├── references.md      ← Enlaces a estándares y herramientas
│   └── stylesheets/       ← CSS personalizado (modo claro/oscuro)
│
├── .opencode/             ← Configuración de agentes OpenCode
│   ├── agents/            ← Definiciones de rol (orchestrator, docwriter, reviewer)
│   ├── commands/          ← Slash commands (/harness-run, /harness-proposal)
│   └── skills/            ← Skills reutilizables (new-doc-page, doc-review)
│
└── openspec/              ← Template para especificaciones (contenido educativo)
```

---

## Uso con OpenCode

```bash
opencode
```

### Comandos disponibles

| Comando | Descripción |
|---|---|
| `/harness-run <tarea>` | Ejecuta el flujo completo para crear/modificar documentación |
| `/harness-proposal <propuesta>` | Define el alcance antes de escribir |

### Flujo de trabajo

1. `/harness-proposal "nueva página sobre X"` → el orquestador analiza y crea una propuesta
2. Revisas y confirmas el plan
3. `/harness-run "nueva página sobre X"` → el orquestador coordina a docwriter + reviewer
4. La documentación se genera y se verifica automáticamente con `uv run zensical build --clean`

---

## Verificación

```bash
uv run zensical build --clean
# Build started
# No issues found
# Build finished in 0.XXs
```

---

## Licencia

MIT

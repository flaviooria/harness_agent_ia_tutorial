---
title: "Introducción al Harness"
icon: lucide/rocket
---

> Guía completa · OpenCode CLI

# Introducción

De hacer preguntas en el chat a tener un orquestador con subagentes especializados trabajando en tu código.
Esta guía cubre todos los artefactos que necesitas crear, explicados desde cero.

`🛠 OpenCode CLI` - `🐍 Python` - `📐 Estándares: AGENTS.md · AgentSkills · OpenSpec`

---

## Fundamentos: Chat IA vs IA Agéntica

!!! note "🎪 Analogía central"
    Imagina que tienes un amigo muy listo. **Chat IA** es llamarle por teléfono: te dice cosas, tú las copias y haces el trabajo. **IA Agéntica** es que ese amigo venga a tu casa: abre el ordenador, escribe el código, lo ejecuta, ve el error, lo corrige, y repite — todo sin que tú intervengas. El **Harness** es el director que coordina a varios de esos amigos especializados.

### ¿Qué es un agente?

Un agente es una IA que opera en un bucle **Percibir → Decidir → Actuar → Observar**. A diferencia de un chat, puede ejecutar código, editar archivos, leer resultados, corregir errores y volver a intentarlo — todo de forma autónoma.

| Dimensión                   | Chat IA                        | IA Agéntica                            |
| --------------------------- | ------------------------------ | -------------------------------------- |
| **Quién ejecuta el código** | Tú, copy-paste                 | El agente, directamente                |
| **El agente, directamente** | Tú copias el error y preguntas | El agente ve el error y se autocorrige |
| **Contexto del proyecto**   | Tú lo explicas cada vez        | El agente lee AGENTS.md y lo tiene     |
| **Coordinación**            | No existe                      | Orquestador delega a subagentes        |
| **Memoria de sesión**       | La conversación                | Archivos en disco, specs persistentes  |

### ¿Qué es un harness?

Un **harness** (arnés) es el sistema que envuelve a los agentes y controla su entorno: qué pueden ver, qué pueden ejecutar, y en qué orden trabajan. En OpenCode, el harness se construye con archivos markdown bien escritos — sin código Python ni scripts adicionales. El harness define el terreno de juego; los agentes juegan dentro de él.

!!! note "OpenCode vs Claude Code"
    Esta guía usa **OpenCode** (open source, go CLI) como base. Todo el sistema de archivos — AGENTS.md, skills, openspec, commands — funciona igual en **Claude Code** (Anthropic). La única diferencia está en el archivo de configuración de MCPs: opencode.json vs .mcp.json. Ambas herramientas leen AGENTS.md de forma nativa. <sup>[[1]](references.md#ref1)</sup> <sup>[[2]](references.md#ref2)</sup>

## El ecosistema completo

Antes de crear ningún archivo, necesitas entender el mapa: qué hace cada pieza y cómo se referencian entre sí.

!!! note "To-do list"
    A partir de ahora vamos a utilizar como ejemplo la construcción de un backend para hacer un proyecto: *`"To-do list"`*

```txt
tu-proyecto/
  │
  ├── AGENTS.md ← reglas maestras, lo lee TODO agente primero
  ├── ARCHITECTURE.md ← mapa del sistema (capas, módulos, decisiones)
  │
  │
  ├── .opencode/ ← configuración local de OpenCode
  │ ├── skills/ ← "cómo hacer" cosas concretas (AgentSkills)
  │ │ ├── fastapi-endpoint/SKILL.md
  │ │ ├── db-migration/SKILL.md
  │ │ └── test-pattern/SKILL.md
  │ └── commands/ ← slash commands reutilizables
  │ ├── harness-run.md
  │ ├── harness-proposal.md
  │ └── harness-review.md
  ├── agents/ ← definición de cada rol del harness
  │ ├── orchestrator.md
  │ ├── backend.md
  │ ├── tester.md
  │ ├── reviewer.md
  │ └── docwriter.md
  │
  ├── openspec/ ← specs del sistema (OpenSpec)
  │ ├── specs/ ← verdad actual ya implementada
  │ │ └── todo-crud/spec.md
  │ └── changes/ ← features en progreso
  │ └── add-priority/
  │ ├── proposal.md
  │ ├── tasks.md
  │
  │
  ├── opencode.json ← MCPs para OpenCode
  └── .mcp.json ← MCPs para Claude Code (mismo contenido)
```

| Documento                  | Responde a…                          | Lo lee…                                        |
| -------------------------- | ------------------------------------ | ---------------------------------------------- |
| `AGENTS.md`                | ¿Cómo se trabaja aquí?               | Todos los agentes, siempre                     |
| `ARCHITECTURE.md`          | ¿Cómo está construido el sistema?    | Orchestrator, reviewer                         |
| `agents/X.md`              | ¿Qué hace este rol?                  | El agente cuando adopta ese rol                |
| `skills/X/SKILL.md`        | ¿Cómo se hace esta tarea concreta?   | El agente indicado en agents del frontmatter   |
| `commands/X.md`            | ¿Qué pasos sigue este slash command? | OpenCode al recibir /comando                   |
| `openspec/specs/X/spec.md` | ¿Qué hace el sistema HOY?            | Orchestrator, reviewer antes de modificar algo |
| `openspec/changes/X/*.md`  | ¿Qué estamos construyendo AHORA?     | Backend, tester durante la implementación      |

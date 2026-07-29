---
title: El Harness
icon: lucide/network
---

## Subagentes

!!! note "⚽ Analogía"
    En un equipo de fútbol, cada posición tiene instrucciones distintas. El portero no sale a rematar córners (aunque podría). Los archivos en agents/ son las instrucciones de cada posición: qué hace, qué no hace, a qué skills recurre, y cuándo para a pedir confirmación.

<div class="grid cards cozy-cards" markdown>

-   :lucide-boxes:{ .lg .middle } __orchestrator__ `Líder`{: .badge .badge-lider }

    ---

    Analiza la tarea, decide qué agentes necesita, los coordina en orden y reúne los resultados.

    * Lee proposal.md + tasks.md
    * Anuncia cada cambio de rol
    * Para si hay ambigüedad

-   :lucide-server:{ .lg .middle } __backend__ `Código`{: .badge .badge-codigo }
 
    ---

    Implementa código de producción siguiendo las capas de ARCHITECTURE.md.

    * Usa skills api-endpoint, db-migration
    * Corre pytest tras cada tarea
    * No escribe tests (eso es tester)

-   :lucide-flask-conical:{ .lg .middle } __tester__ `Tests`{: .badge .badge-tests }

    ---

    Escribe y ejecuta tests unitarios e integración para el código del backend.

    * Usa skill test-pattern
    * Cubre: feliz + error + edge case
    * Reporta fallos con contexto

-   :lucide-eye:{ .lg .middle } __reviewer__ `Calidad`{: .badge .badge-calidad }
 
    ---

    Revisa calidad, seguridad y adherencia a capas antes de abrir el PR.

    * Usa MCP postgres para verificar esquema
    * Clasifica: blocker / warning / suggestion
    * No escribe código nuevo

-  :lucide-file-text:{ .lg .middle } __docwriter__ `Docs`{: .badge .badge-docs }
    
    ---

    Escribe docstrings, README y descripciones OpenAPI para el código del backend.

    * Usa MCP github para abrir PRs
    * No escribe código nuevo
    * No corrige errores de backend

</div>

## Estructura estándar de un agent file

Todos los archivos de `.opencode/agents/` siguen el mismo patrón: cabecera YAML opcional + secciones markdown bien definidas.

> .opencode/agents/orchestrator.md

```markdown
---
name: orchestrator
role: Coordina a los demás agentes del harness
---

# Agent: orchestrator

## Responsabilidad
Analizar la tarea recibida, decidir qué agentes se necesitan,
ejecutarlos en el orden correcto, y reportar el resultado al usuario.

## Protocolo de orquestación

Paso 1: Analizar
- Lee el change en openspec/changes/ si existe.
- Lee ARCHITECTURE.md para entender el impacto.
- Si no hay spec → créalo con /harness-proposal y espera confirmación.

Paso 2: Decidir qué agentes activan
No siempre se necesitan todos. Ejemplos:
- Bug fix menor:         backend → tester
- Feature nueva:         backend → tester → reviewer → docwriter
- Solo refactor:         backend → reviewer
- Solo documentación:    docwriter

Paso 3: Ejecutar con anuncios de rol
Antes de cada cambio de agente, escribe:
--- [AGENTE: backend] Implementando tarea 1/4 ---
Luego actúa como ese agente leyendo su archivo agents/{nombre}.md.

Paso 4: Reportar
Al terminar: resumen de cambios, tests pasados/fallados,
y enlace al change en openspec/.

## Cuándo parar y pedir confirmación
- Antes de tocar migrations o models de producción.
- Si el spec es ambiguo en más de un punto.
- Si pytest falla más de 2 veces seguidas en el mismo test.
- Si el reviewer reporta un blocker.

## Qué NO hace el orchestrator
No implementa código directamente. Delega siempre.
No salta al reviewer si los tests no están en verde.
```

> .opencode/agents/backend.md

```markdown
---
name: backend
role: Implementa código de producción
triggers:
  - nuevo endpoint
  - cambio de modelo
  - refactor de servicio
  - nueva feature
---

# Agent: backend

## Responsabilidad
Implementar código de producción siguiendo las capas definidas
en ARCHITECTURE.md, usando las skills correspondientes.

## Skills que usa
- .opencode/skills/fastapi-endpoint/SKILL.md  (endpoints nuevos)
- .opencode/skills/db-migration/SKILL.md     (cambios de esquema)

## Protocolo de implementación
1. Leer el spec del change en openspec/changes/{nombre}/spec.md.
2. Leer la skill relevante antes de escribir una sola línea.
3. Implementar tarea por tarea según tasks.md.
4. Ejecutar uv run pytest después de cada tarea sustancial.
5. Marcar [x] en tasks.md al completar cada ítem.

## Qué NO hace
- No escribe tests (responsabilidad de tester.md).
- No revisa código de otros (responsabilidad de reviewer.md).
- No crea documentación (responsabilidad de docwriter.md).
```

## Skills — El "cómo hacer" con AgentSkills

!!! note "📋 Analogía"
    Las skills son recetas de cocina específicas para tu restaurante. No "cómo se hace una tortilla en general" — sino "cómo se hace una tortilla en ESTE restaurante, con ESTE fuego, en ESTA sartén, con ESTOS ingredientes". El agente sigue la receta exacta sin improvisar.

Una skill es un archivo markdown que le enseña al agente cómo ejecutar un patrón repetible en tu proyecto específico. Se consulta antes de implementar, no después. El estándar AgentSkills define el formato con frontmatter YAML para que los agentes puedan descubrirlas por trigger. <sup>[[4]](references.md#ref4)</sup>

### Estructura estándar (AgentSkills)

> .opencode/skills/fastapi-endpoint/SKILL.md

```markdown
    ---
    name: fastapi-endpoint
    version: "1.0"
    description: >
    Usar cuando se cree o modifique un endpoint FastAPI en este proyecto.
    Define el patrón exacto router→service→repository.
    triggers:
    - nuevo endpoint
    - crear ruta HTTP
    - añadir API
    - POST GET PUT PATCH DELETE
    agents:
    - backend
    - reviewer
    ---

    # Skill: FastAPI endpoint

    ## Cuándo usar esta skill
    Cualquier vez que se añada o modifique una ruta HTTP
    (@router.get, @router.post, etc.) en este proyecto.

    ## Patrón de capas OBLIGATORIO

    app/routers/{recurso}.py     ← solo HTTP: valida, enruta, devuelve schema
    app/services/{recurso}.py    ← lógica de negocio, sin imports de FastAPI
    app/repositories/{recurso}.py← queries SQLAlchemy async
    app/schemas/{recurso}.py     ← Pydantic v2 I/O

    El router NUNCA contiene SQL ni lógica de negocio.
    Si ves eso en código existente, es deuda técnica: corrígelo si tocas ese fichero.

    ## Plantilla de router

    ```python
    # app/routers/todos.py
    from fastapi import APIRouter, Depends, HTTPException
    from app.schemas.todo import TodoCreate, TodoRead
    from app.services.todo import TodoService
    from app.dependencies import get_todo_service

    router = APIRouter(prefix="/todos", tags=["todos"])

    @router.post("", response_model=TodoRead, status_code=201)
    async def create_todo(
        payload: TodoCreate,
        service: TodoService = Depends(get_todo_service),
    ) -> TodoRead:
        """Create a new todo item."""
        return await service.create(payload)

    @router.get("/{todo_id}", response_model=TodoRead)
    async def get_todo(
        todo_id: int,
        service: TodoService = Depends(get_todo_service),
    ) -> TodoRead:
        """Get a single todo by ID."""
        todo = await service.get_by_id(todo_id)
        if not todo:
            raise HTTPException(status_code=404, detail="Todo not found")
        return todo
    ```

    ## Manejo de errores
    - 404: recurso no encontrado →
    raise HTTPException(404, detail="Todo not found")
    - 422: lo gestiona Pydantic automáticamente. No duplicar.
    - Errores de dominio: usar una excepción propia en el service
    capturada por un handler global en app/main.py.

    ## Checklist antes de dar el endpoint por terminado
    - [ ] Schema Pydantic de entrada y de salida están definidos.
    - [ ] El router no contiene SQL ni lógica de negocio.
    - [ ] Existe al menos un test de happy path y uno de error (404 o 422).
    - [ ] El endpoint tiene summary= si el nombre no es autoexplicativo.
    - [ ] El status code es correcto: 200 GET, 201 POST, 204 DELETE sin body.

    ## Referencias relacionadas
    - Para cambios en el modelo de datos: .opencode/skills/db-migration/SKILL.md
    - El comportamiento esperado debe estar en openspec/specs/{recurso}/spec.md
```

> .opencode/skills/db-migration/SKILL.md

```markdown
    ---
    name: db-migration
    version: "1.0"
    description: >
    Usar cuando se añada o modifique el esquema de la base de datos.
    Cubre modificación de modelo, generación de migración Alembic,
    revisión manual y verificación post-migración.
    triggers:
    - nueva columna
    - nueva tabla
    - cambio de esquema
    - migración
    - alembic
    agents:
    - backend
    ---

    # Skill: Migración de base de datos

    ## Pasos OBLIGATORIOS

    1. Modifica el modelo
    En app/models/{recurso}.py. El modelo es la fuente de verdad.

    2. Genera la migración
    ```bash
    uv run alembic revision --autogenerate -m "descripcion_snake_case"
    ```

    3. REVISA el archivo generado manualmente (crítico)
    Alembic NO detecta bien:
    - Renombrados de columna (genera DROP + ADD, pierdes datos).
    - Cambios de tipo complejos.
    - Constraints nombradas.
    Abre el archivo en alembic/versions/ y verifica cada operación.

    4. Aplica en local
    ```bash
    uv run alembic upgrade head
    ```

    5. Verifica con MCP de PostgreSQL
    Usa el MCP para ejecutar:
    \d todos  (psql)  o  describe table todos
    Confirma que las columnas, tipos e índices son correctos.

    ## Reglas importantes
    - NUNCA usar Base.metadata.create_all() en producción.
    - Toda columna NOT NULL en tabla con datos existentes necesita
    server_default o un paso de backfill explícito.
    - Nombrar la migración igual que el change de OpenSpec que la originó.

    ## Ejemplo: columna NOT NULL con backfill seguro
    ```python
    # En el archivo de migración generado
    def upgrade():
        # Paso 1: añadir como nullable con default
        op.add_column("todos", sa.Column(
            "priority", sa.String(), nullable=False, server_default="medium"
        ))
        # Paso 2 (si no quieres mantener el default permanente):
        # op.alter_column("todos", "priority", server_default=None)
    ```
```

## Commands — Slash commands reutilizables

!!! note "📋 Analogía"
    Los commands son atajos de teclado que ejecutan un patrón repetible. No "cómo se hace una tortilla en general" — sino "cómo se hace una tortilla en ESTE restaurante, con ESTE fuego, en ESTA sartén, con ESTOS ingredientes". El agente sigue el comando exacto sin improvisar.

Los commands viven en .opencode/commands/. Cada archivo markdown es un slash command que OpenCode registra automáticamente.<sup>[[5]](references.md#ref5)</sup> La variable $ARGUMENTS recibe el texto que escribas después del nombre del comando.

> .opencode/commands/harness-run.md

```markdown
    ---
    description: Ejecuta el flujo completo del harness para una tarea nueva
    usage: /harness-run 
    example: /harness-run añadir campo priority (high/medium/low) a los todos
    ---

    La tarea a implementar es: "$ARGUMENTS"

    Ejecuta el flujo completo del harness siguiendo estos pasos:

    1. ANALIZAR (siempre primero)
    Lee agents/orchestrator.md.
    Busca en openspec/changes/ si ya existe un spec para esta tarea.
    Si no existe: PARA y crea el spec con /harness-proposal antes de continuar.

    2. PLANIFICAR
    Lee el proposal.md y tasks.md del change.
    Decide qué agentes se necesitan según el tipo de tarea.
    Anuncia el plan al usuario antes de implementar.

    3. BACKEND (si aplica)
    Anuncia: --- [AGENTE: backend] ---
    Lee agents/backend.md y las skills relevantes.
    Implementa tarea por tarea de tasks.md.
    Corre uv run pytest tras cada tarea. Si falla, corrige antes de seguir.

    4. TESTER (si aplica)
    Anuncia: --- [AGENTE: tester] ---
    Lee agents/tester.md y .opencode/skills/test-pattern/SKILL.md.
    Escribe tests para los escenarios del spec que no estén cubiertos.

    5. REVIEWER (si aplica)
    Anuncia: --- [AGENTE: reviewer] ---
    Lee agents/reviewer.md y ARCHITECTURE.md.
    Usa MCP de postgres para verificar el esquema si hubo migración.
    Lista findings con severidad: BLOCKER / WARNING / SUGGESTION.
    Si hay BLOCKERS: para y reporta antes de continuar.

    6. DOCWRITER (si aplica)
    Anuncia: --- [AGENTE: docwriter] ---
    Actualiza docstrings, descripciones OpenAPI y CHANGELOG si aplica.

    7. REPORTE FINAL
    Resumen: cambios implementados, tests en verde, issues del reviewer,
    y ruta al spec en openspec/.
```

> .opencode/commands/harness-proposal.md

```markdown
    ---
    description: Crea un OpenSpec (proposal + tasks + spec delta) antes de codear
    usage: /harness-proposal 
    example: /harness-proposal añadir campo due_date a los todos
    ---

    El cambio propuesto es: "$ARGUMENTS"

    Sigue estos pasos (sin escribir código):

    1. BUSCAR CONTEXTO
    Lee openspec/specs/ para encontrar specs relacionados con la tarea.
    Lee el código existente en app/ relacionado con el área afectada.

    2. CREAR EL CHANGE
    Determina un nombre corto en kebab-case para el change.
    Crea la carpeta openspec/changes/{nombre}/ con:

    proposal.md — estructura:
    ## Why: por qué se necesita este cambio
    ## What changes: qué se modifica (campos, endpoints, comportamiento)
    ## Impact: qué specs afecta, si rompe compatibilidad

    tasks.md — checklist de implementación:
    - [ ] N. descripción de tarea concreta

    specs/{spec-afectado}/spec.md — delta del spec:
    Nuevos Requirements con Scenarios en formato Gherkin
    (GIVEN / WHEN / THEN).

    3. MOSTRAR RESUMEN
    Presenta el plan creado y espera confirmación antes de implementar.
    No escribas una sola línea de código en este paso.
```
## Specs — El estándar OpenSpec

!!! note "📋 Analogía"
    No contratas a un arquitecto y le dices "haz algo bonito". Le das los planos: cuántas habitaciones, dónde las puertas, qué altura. Un spec es el plano. Sin plano, el agente construye lo que interpreta — que puede no ser lo que querías.

OpenSpec es un framework de especificación ligero mantenido por Fission AI. Las specs viven en el repositorio junto al código — no desaparecen cuando termina una sesión de chat. Cada feature pasa por dos estados: change en progreso (openspec/changes/) y spec archivado (openspec/specs/). <sup>[[6]](references.md#ref6)</sup>

1. **Proponer** (`/harness-proposal`)
    - Creas `openspec/changes/{nombre}/proposal.md` + `tasks.md` + `specs/{afectado}/spec.md`. Sin código todavía. Revisas y apruebas el plan.
2. **Implementar** (`/harness-run`)
    - El harness lee el `tasks.md` y lo ejecuta tarea por tarea, marcando casillas a medida que avanza.
3. **Archivar** (`/harness-archive`)
    - Cuando todo está en verde, el harness mueve el spec delta a `openspec/specs/{afectado}/spec.md` y borra el change.

### Ejemplo completo: añadir campo priority

> openspec/changes/add-priority/proposal.md

```markdown
# Change: add-priority

## Why
Los usuarios no pueden priorizar tareas. Necesitan ordenar por urgencia.
Esta feature es la más pedida en el backlog.

## What changes
- Columna priority (enum: high / medium / low) en tabla todos.
- NOT NULL, default: medium.
- POST /todos y PATCH /todos/{id} aceptan el campo priority.
- GET /todos acepta parámetro ?priority=high para filtrar.

## Impact
- Afecta al spec: todo-crud
- Requiere migración Alembic (skill: db-migration)
- Backward compatible: el campo tiene default, no rompe clientes existentes
- No afecta autenticación ni otros recursos
```

> openspec/changes/add-priority/tasks.md

```markdown
- [ ] 1. Añadir campo priority: str al modelo Todo
        con server_default="medium" y nullable=False
- [ ] 2. Migración Alembic: add_priority_to_todos
        (usar skill db-migration, revisar manualmente)
- [ ] 3. Actualizar schemas: TodoCreate, TodoUpdate (optional),
        TodoRead (include priority)
- [ ] 4. Parámetro ?priority= en endpoint GET /todos
        con filtro en el repository
- [ ] 5. Tests para los 3 escenarios del spec delta
        (skill: test-pattern)
- [ ] 6. uv run pytest en verde
- [ ] 7. uv run ruff check . sin errores
- [ ] 8. Abrir PR referenciando este change
- [ ] 9. Cuando todo esté en verde, archivar el spec delta en openspec/specs/todo-crud/spec.md
```
```

> openspec/changes/add-priority/specs/todo-crud/spec.md

```markdown
# todo-crud Specification (delta: add-priority)

## Requirement: Crear tarea con prioridad
El sistema SHALL aceptar un campo priority en la creación de tareas.
Los valores válidos son: high, medium, low.

#### Scenario: Prioridad explícita
- GIVEN un usuario autenticado
- WHEN envía POST /todos con {"title": "Urgente", "priority": "high"}
- THEN se crea la tarea con priority=high
- AND se devuelve 201 con el recurso creado incluyendo priority

#### Scenario: Prioridad por defecto
- GIVEN un usuario autenticado
- WHEN envía POST /todos con {"title": "Normal"} (sin priority)
- THEN se crea la tarea con priority="medium"
- AND se devuelve 201

#### Scenario: Valor de prioridad inválido
- GIVEN un usuario autenticado
- WHEN envía POST /todos con {"title": "X", "priority": "urgent"}
- THEN se devuelve 422 con mensaje de validación
- AND el body especifica que priority debe ser high/medium/low

## Requirement: Filtrar tareas por prioridad
El sistema SHALL permitir listar solo tareas de una prioridad concreta.

#### Scenario: Filtro por prioridad high
- GIVEN existen tareas con priority=high y priority=medium
- WHEN envía GET /todos?priority=high
- THEN solo se devuelven las tareas con priority=high
```

### El spec archivado: la fuente de verdad permanente
Una vez que el PR está mergeado, el spec delta se integra al spec permanente en `openspec/specs/`. Esta es la documentación viva del sistema — siempre más actualizada que cualquier wiki o Notion. 

> openspec/specs/todo-crud/spec.md

```markdown
# todo-crud Specification
## Purpose
Gestionar el ciclo de vida de una tarea: crear, leer, actualizar,
completar y filtrar.

## Requirement: Crear una tarea
El sistema SHALL crear tareas con título obligatorio,
descripción opcional y prioridad (high/medium/low, default: medium).

#### Scenario: Creación mínima válida
- GIVEN un usuario autenticado
- WHEN envía POST /todos con {"title": "Comprar leche"}
- THEN se crea con completed=false y priority="medium"
- AND se devuelve 201

[... más scenarios ...] 

## Requirement: Completar una tarea
[...] 

## Requirement: Filtrar por prioridad
[...] 

## Historial de cambios
- 2026-01-15: spec inicial (CRUD básico)
- 2026-02-03: add-priority (campo priority con filtro)
```

---
title: Referencias
icon: lucide/book
---

## Arquitectura del Harness

El harness coordina subagentes especializados que trabajan en cadena. Cada subagente lee su propia definición en `agents/`, usa skills para tareas concretas, y se conecta a servicios externos mediante MCP.

```mermaid
graph TD
    User["👤 Tú / Usuario"] -->|"/harness-run"| Orch["🤖 Orchestrator"]

    Orch -->|Lee| ARCH["📋 ARCHITECTURE.md<br/>Estructura del proyecto"]
    Orch -->|Lee| AGENTS["📋 AGENTS.md<br/>Reglas del proyecto"]
    Orch -->|Lee| AgentFiles["📁 agents/*.md<br/>Definiciones de rol"]

    subgraph Subagentes["Subagentes del Harness"]
        direction TB
        Backend["⚙️ backend"]
        Tester["🧪 tester"]
        Reviewer["🔍 reviewer"]
        DocWriter["📝 docwriter"]
    end

    Orch -->|Delega en orden| Backend
    Backend -->|Verifica| Tester
    Tester -->|Revisa| Reviewer
    Reviewer -->|Documenta| DocWriter

    subgraph Skills["📚 Skills (.opencode/skills/)"]
        SkillAPI["fastapi-endpoint"]
        SkillDB["db-migration"]
        SkillTest["test-pattern"]
    end

    Backend -->|Usa| SkillAPI
    Backend -->|Usa| SkillDB
    Tester -->|Usa| SkillTest
    Reviewer -->|Usa| SkillAPI
    Reviewer -->|Usa| SkillDB

    subgraph MCP["🌐 MCP Servers"]
        MCP_DB[("🗄️ PostgreSQL")]
        MCP_GH["🐙 GitHub"]
        MCP_FS["📁 File System"]
    end

    Reviewer -->|Consulta esquema| MCP_DB
    Backend -->|Ejecuta migraciones| MCP_DB
    Tester -->|Verifica datos| MCP_DB
    DocWriter -->|Abre PR| MCP_GH
    Reviewer -->|Comenta en PR| MCP_GH
    User -->|Configura| MCP

    DocWriter -->|Reporta| Orch
    Reviewer -->|Reporta| Orch
    Tester -->|Reporta| Orch
    Backend -->|Reporta| Orch

    Orch -->|Resumen final| User

    style User fill:#e1f5fe,stroke:#0288d1,color:#01579b
    style Orch fill:#fff3e0,stroke:#f57c00,color:#e65100
    style Subagentes fill:#f5f5f5,stroke:#9e9e9e,stroke-dasharray: 5 5,color:#424242
    style Skills fill:#f3e5f5,stroke:#9c27b0,stroke-dasharray: 5 5,color:#4a148c
    style MCP fill:#e8f5e9,stroke:#4caf50,stroke-dasharray: 5 5,color:#1b5e20
```

## Protocolo MCP — Model Context Protocol

### ¿Qué es MCP?

MCP (Model Context Protocol) es el estándar abierto que permite a los agentes conectarse con herramientas y servicios externos. Piensa en MCP como un USB para IAs: un mismo conector funciona con bases de datos, APIs, sistemas de archivos, y más. <sup>[[7]](references.md#ref7)</sup>

Sin MCP, un agente solo puede leer y escribir archivos en tu repositorio. Con MCP, puede:

- Consultar una base de datos PostgreSQL para verificar esquemas
- Crear PRs en GitHub automáticamente
- Ejecutar queries SQL para depurar datos
- Leer issues y comentarios en GitHub
- Navegar por el sistema de archivos fuera del proyecto

### Cómo funciona el flujo MCP

<center>
    <img src="./assets/workflow_mcp.png" alt="Flujo MCP" width="600"/>
</center>

### Servidores MCP típicos en un harness

| Servidor MCP | Para qué lo usa | Agente que lo activa |
|---|---|---|
| **PostgreSQL** | Consultar esquema de tablas, verificar migraciones, depurar datos | reviewer, backend, tester |
| **GitHub** | Crear PRs, añadir comentarios, leer issues | docwriter, reviewer |
| **File System** | Leer/escribir archivos fuera del proyecto | docwriter, orchestrator |

### Configuración

Los MCPs se configuran en `opencode.json` (OpenCode) o `.mcp.json` (Claude Code):

```json
{
  "mcpServers": {
    "postgres": {
      "command": "uvx",
      "args": [
        "mcp-postgres",
        "--connection-string",
        "postgresql://user:pass@localhost:5432/todos"
      ]
    },
    "github": {
      "command": "uvx",
      "args": [
        "mcp-github",
        "--token",
        "${GITHUB_TOKEN}"
      ]
    }
  }
}
```

!!! tip "MCP en acción"
    En el flujo de trabajo de abajo, el reviewer usa `MCP postgres: \d todos` para verificar que la migración creó la columna `due_date` correctamente. El agente no *adivina* si el esquema es correcto — lo *verifica* contra la BD real vía MCP.

## Flujo de trabajo completo

Con todo configurado, una sesión de trabajo real se ve así:

- Terminal — OpenCode session

```
 Abres OpenCode en tu proyecto
cd todo-harness
opencode

# Propones una feature nueva (sin código todavía)
tú:  /harness-proposal añadir campo due_date (fecha límite) a los todos

orchestrator:
  Buscando specs relacionados en openspec/specs/...
  Leyendo openspec/specs/todo-crud/spec.md...
  Revisando código existente en app/models/todo.py...

  Creando openspec/changes/add-due-date/proposal.md ✓
  Creando openspec/changes/add-due-date/tasks.md    ✓
  Creando openspec/changes/add-due-date/specs/...   ✓

  Plan creado. Resumen:
  - 8 tareas de implementación
  - Afecta spec: todo-crud
  - Requiere migración Alembic
  - Backward compatible (campo nullable)

  ¿Implemento?

# Revisas el plan en openspec/changes/add-due-date/ y apruebas
tú:  sí, implementa

orchestrator:
  --- [AGENTE: backend] Implementando 8 tareas ---

  Tarea 1: Añadiendo campo due_date al modelo Todo...
  Tarea 2: Generando migración Alembic...
  → Revisando migración generada manualmente...
  Tarea 3: Aplicando migración...
  Tarea 4: Actualizando schemas...
  Tarea 5: Añadiendo filtro ?overdue=true...
  Ejecutando: uv run pytest... ✓ 14 tests pasados

  --- [AGENTE: tester] Revisando cobertura ---

  Leyendo spec delta para escenarios no cubiertos...
  Añadiendo test: test_create_with_due_date ✓
  Añadiendo test: test_filter_overdue ✓
  Ejecutando: uv run pytest -v... ✓ 16 tests pasados

  --- [AGENTE: reviewer] Revisando calidad ---

  Verificando capas... ✓
  Verificando type hints... ✓
  Usando MCP postgres: \d todos...
    → Columna due_date presente: TIMESTAMP nullable ✓
  Findings:
    SUGGESTION: añadir índice en due_date para queries de overdue

  --- [AGENTE: docwriter] ---

  Actualizando docstrings de endpoints afectados ✓
  Añadiendo entrada en CHANGELOG ✓

  Resumen final:
  ✓ 16 tests en verde
  ✓ Migración verificada en BD
  ✓ Sin blockers ni warnings
  → Suggestion pendiente: índice en due_date
  → Listo para PR
```

!!! info "🎯 Lo que acaba de pasar"
    El orquestador coordinó 4 agentes distintos, cada uno leyendo sus propias instrucciones, usando skills especializadas, verificando el esquema real de la BD vía MCP, y produciendo un output coherente — todo desde una sola instrucción tuya. Eso es la diferencia entre chat IA e IA agéntica.


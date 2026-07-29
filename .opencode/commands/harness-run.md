---
description: Ejecuta el flujo completo del harness para documentación nueva
usage: /harness-run 
example: /harness-run crear página sobre agent skills
---

La tarea a implementar es: "$ARGUMENTS"

Ejecuta el flujo completo del harness siguiendo estos pasos:

1. ANALIZAR (siempre primero)
Lee `.opencode/agents/orchestrator.md`.
Lee `ARCHITECTURE.md` para entender la estructura del proyecto.
Revisa los archivos en `docs/` para conocer el contenido existente.
Lee la skill relevante en `.opencode/skills/` antes de implementar.

mem_current_project → confirma el proyecto detectado.
mem_search "$ARGUMENTS" → busca contexto previo relacionado con la tarea.
Si hay resultados relevantes: úsalos como punto de partida.

Si la tarea no está definida: PARA y créala con `/harness-proposal` antes de continuar.

2. PLANIFICAR
Define el alcance del contenido: qué páginas crear o modificar.
Decide qué agentes se necesitan según el tipo de tarea.
Anuncia el plan al usuario antes de implementar.

3. IMPLEMENTACIÓN
Anuncia: --- [AGENTE: docwriter](../agents/docwriter.md) ---
Lee la skill `new-doc-page` en `.opencode/skills/new-doc-page/SKILL.md`.
Implementa los ficheros markdown para zensical en `docs/` siguiendo las convenciones del proyecto.
Si se descubre algo no obvio durante la implementación:
  mem_save(title, type="discovery", content="What/Why/Where/Learned")

4. REVISAR
Anuncia: --- [AGENTE: reviewer](../agents/reviewer.md) ---
Lee la skill `doc-review` en `.opencode/skills/doc-review/SKILL.md`.
Revisa los ficheros generados y corrige errores de redacción, ortografía y estilo.
Ejecuta `uv run zensical build --clean` para verificar que buildée sin errores.

5. REPORTE FINAL
mem_session_summary(content="
  ## Goal
  [qué se hizo]
  ## Discoveries
  [lo no obvio que se descubrió]
  ## Accomplished
  [cambios completados]
  ## Next Steps
  [qué queda pendiente]
  ## Relevant Files
  [archivos tocados]
")
Resumen: cambios implementados, issues del reviewer, y ruta a los archivos en `docs/`.
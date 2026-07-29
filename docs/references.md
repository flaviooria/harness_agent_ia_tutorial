---
title: Referencias
icon: lucide/scroll-text
---

# Referencias

Todos los estándares y herramientas referenciados en esta guía están documentados en las siguientes fuentes. Se recomienda consultarlas para la versión más actualizada de cada especificación.

---

<div class="cozy-references" markdown>

1. <span id="ref1"></span>__OpenCode — Documentación oficial__  
   CLI open source para coding agéntico. Leer en especial las secciones de *Rules* (equivalente a AGENTS.md) y *MCP configuration*.  
   <https://opencode.ai/docs>

2. <span id="ref2"></span>__Anthropic — Claude Code documentation__  
   Documentación oficial de Claude Code. Cubre memory (CLAUDE.md), sub-agents (Task tool), MCPs y slash commands.  
   <https://docs.anthropic.com/en/docs/claude-code>

3. <span id="ref3"></span>__AGENTS.md — Formato abierto__  
   Estándar abierto mantenido por la Agentic AI Foundation (Linux Foundation). Incluye FAQ, ejemplos de proyectos reales (*apache/airflow*, *openai/codex*) y guía de migración desde AGENT.md / CLAUDE.md.  
   <https://agents.md/> · <https://github.com/agentsmd/agents.md>

4. <span id="ref4"></span>__AgentSkills — Estándar de skills para agentes__  
   Especificación del formato de skills reutilizables para agentes de código. Define el frontmatter YAML (*name*, *version*, *description*, *triggers*, *agents*) y el sistema de auto-descubrimiento.  
   <https://agentskills.io/home>

5. <span id="ref5"></span>__OpenCode — Custom slash commands__  
   Documentación de cómo crear y usar comandos personalizados en OpenCode. Los comandos van en `.opencode/commands/` y se registran automáticamente.  
   <https://opencode.ai/docs/commands>

6. <span id="ref6"></span>__OpenSpec — Framework de spec-driven development__  
   Framework ligero para especificación de features. Specs en el repositorio, integración nativa con OpenCode (`/openspec:proposal`), Cursor, Claude Code y otros. Mantenido por Fission AI.  
   <https://openspec.dev/> · <https://github.com/Fission-AI/OpenSpec>

7. <span id="ref7"></span>__Model Context Protocol (MCP) — Anthropic__  
   Protocolo estándar para conectar LLMs con herramientas externas. Incluye lista de servidores oficiales (*postgres*, *github*, *filesystem*, *slack*, *brave-search*, *puppeteer* y muchos más).  
   <https://modelcontextprotocol.io> · <https://github.com/modelcontextprotocol/servers>

8. <span id="ref8"></span>__Anthropic — Building effective agents__  
   Artículo de referencia sobre patrones de diseño para sistemas multi-agente: orquestador/subagente, tool use, agentic loops. Lectura obligatoria antes de diseñar un harness.  
   <https://www.anthropic.com/research/building-effective-agents>

9. <span id="ref9"></span>__Anthropic — Prompt engineering guide__  
   Guía oficial para escribir prompts efectivos. Aplicable directamente al diseño de agent files, skills y commands: uso de XML, chain-of-thought, ejemplos positivos y negativos.  
   <https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview>

10. <span id="ref10"></span>__skills.sh — Explorador de skills__  
    Directorio comunitario de skills reutilizables para distintos stacks (FastAPI, Django, React, Go...). Útil para inspirarse en el formato y contenido de SKILL.md antes de escribir los tuyos.  
    <https://www.skills.sh/>

11. <span id="ref11"></span>__Agentic AI Foundation (AAIF)__  
     Organismo bajo la Linux Foundation que mantiene el estándar AGENTS.md y coordina la compatibilidad entre herramientas agénticas.  
     <https://aaif.io>

12. <span id="ref12"></span>__Engram — Memoria persistente para agentes IA__  
     Binary Go con SQLite + FTS5, MCP server, HTTP API, CLI y TUI. Agent-agnostic, zero dependencies. Open source por Gentleman Programming.  
     <https://github.com/Gentleman-Programming/engram> · <https://github.com/Gentleman-Programming/engram/blob/main/DOCS.md>

</div>
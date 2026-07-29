---
description: Define el alcance de una página o sección de documentación antes de escribirla
usage: /harness-proposal 
example: /harness-proposal página de introducción a skills
---

La propuesta de documentación es: "$ARGUMENTS"

Sigue estos pasos (sin escribir documentación):

1. BUSCAR CONTEXTO
Lee los archivos existentes en `docs/` para entender el contenido actual y evitar duplicados.
Lee `ARCHITECTURE.md` para conocer la estructura del proyecto.

mem_search "$ARGUMENTS" → verifica que no existe una propuesta o decisión similar previa.
Si hay resultados: revisa si ya está cubierto o si se puede reutilizar.

2. CREAR LA PROPUESTA
Determina un nombre corto en kebab-case para la propuesta.
Define la estructura del contenido con:

propuesta.md — estructura:
  ## Why: por qué se necesita esta página o sección
  ## What changes: qué se crea o modifica (secciones, páginas, contenido)
  ## Impact: cómo se relaciona con el resto de la documentación

tasks.md — checklist de implementación:
  - [ ] N. descripción de tarea concreta

Contenido propuesto:
  Esquema de la página: frontmatter YAML (title, icon), secciones,
  ejemplos de código, enlaces a otras páginas del sitio.

3. MOSTRAR RESUMEN
Presenta el plan creado y espera confirmación antes de implementar.
No escribas una sola línea de documentación en este paso.
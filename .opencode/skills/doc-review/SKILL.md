---
name: doc-review
description: Revisar calidad de páginas de documentación
agents: [reviewer]
---

# Skill: doc-review

Checklist para revisar páginas de documentación.

## Blocker (debe corregirse)

- [ ] El frontmatter YAML tiene `title` e `icon`
- [ ] El icono usa el prefijo `lucide/` con un nombre válido
- [ ] El archivo sigue la convención `snake_case.md`
- [ ] Los enlaces internos apuntan a rutas que existen
- [ ] `uv run zensical build --clean` pasa sin errores

## Warning (debería corregirse)

- [ ] No hay errores de ortografía en español
- [ ] La redacción es clara y consistente
- [ ] Los ejemplos de código tienen el lenguaje especificado (```python)
- [ ] Las admonitions tienen título descriptivo
- [ ] El tono es consistente con el resto de la guía

## Suggestion (opcional)

- [ ] La página podría beneficiarse de ejemplos adicionales
- [ ] Algún enlace externo podría ser útil
- [ ] La navegación entre páginas relacionadas es fluida

---
id: GOV-004-E
status: in_review
---

# Session E — Agents and skills

```text
Rol: ingeniero de plataforma experto en Claude Code (subagents en .claude/agents/, skills en .claude/skills/,
settings, hooks). Háblame en español; archivos en inglés.

Contexto: lee ../CLAUDE.md, cherryLove-docs/CLAUDE.md, STATUS.md (verifica requisitos approved y ADRs
de C approved; si no, detente), decision-log.md, 04-architecture/ y 09-agents-skills/README.md (propuesta v0).
Consulta la documentación oficial actual de Claude Code antes de definir formatos.

Objetivo:
1. Revisa la propuesta v0 contra los ADRs aprobados y proponme la lista final de agentes (p. ej. analyst,
   architect, ux, django-dev, react-dev, rn-dev, qa, security-reviewer, teacher, docs-keeper) y skills
   (approval gate, ports-adapters, módulo Django, módulo React, paridad RN, sync, seguridad, notebook,
   commit/PR/evidencia), con responsabilidad, disparador, herramientas permitidas y en qué repo viven.
2. Tras mi aprobación, redacta cada uno. Los skills codifican ADRs aprobados citando sus IDs, nunca opiniones nuevas.
3. Propón .claude/settings.json por repo con permisos mínimos (deny para .env y secretos).
4. Prueba cada skill con un caso real y documenta el resultado en 08-evidence/.

Reglas: un skill no puede contradecir un DEC/ADR; nada se instala sin mi aprobación.
```

---
id: GOV-004-A
status: in_review
---

# Session A — Requirements freeze

```text
Rol: analista de software senior, obsesivo con el detalle. Háblame en español; los archivos en inglés.

Contexto: lee ../CLAUDE.md y cherryLove-docs/CLAUDE.md, luego STATUS.md, 00-governance/decision-log.md,
glossary.md y todo 02-requirements/.

Objetivo: cerrar el freeze de requerimientos.
1. Resuelve conmigo las preguntas abiertas de open-questions.md (hoy: OQ-001), una por una, con tu recomendación.
2. Revisa functional.md, nfr.md y roles-actuators.md buscando: contradicciones con el decision-log, filas
   marcadas "— (needs freeze)", huecos (estados vacíos, errores, límites, borrados en cascada) y requisitos
   no verificables. Preséntame cada hallazgo con ID afectado + recomendación y pregúntame.
3. Aplica solo lo que yo apruebe. Cada decisión nueva = fila DEC nueva.
4. Cuando yo diga "freeze": marca REQ-DOC-001…004 approved con fecha, actualiza STATUS.md y haz commit
   `docs(requirements): freeze v1` citando los DEC.

Reglas: nunca asumas; no avances a UX, ADRs ni código; no edites decisiones existentes.
Entregable: requisitos aprobados + STATUS.md actualizado + commit.
```

---
id: GOV-004-F
status: in_review
---

# Session F — Implementation plan: API (`cherrylove-api`)

```text
Rol: arquitecto de software senior y profesor. Explícame el porqué de cada fase, recomienda una opción y espera
mi aprobación. Háblame en español; archivos en inglés.

Precondiciones (verifícalas y detente si falta alguna): REQ-DOC-001…004, C4-001, ADR-001…016, DATA-001 y API-001
en approved (STATUS.md). Lee ../CLAUDE.md, cherryLove-docs/CLAUDE.md, decision-log.md, glossary.md,
02-requirements/ y 04-architecture/. Solo escribes en 06-implementation/api/.

Objetivo: plan de implementación por fases de cherrylove-api (1 PR por fase, 1 commit por paso, ≥ 1 test por
paso, cada paso cita sus REQ/DEC/ADR; DEC-023, ADR-014, ADR-015).
1. Re-verifica en fuentes oficiales las versiones de ADR-016 (REQ-N-QA-05) y lista los bumps de patch.
2. Fase 0 (fundaciones): uv, Docker Compose (PostgreSQL, Redis, mail catcher, storage S3), CI de ADR-015,
   import-linter, roles de BD app/maintenance (ADR-008 §4), y como PRIMER paso el spike de DEC-037: elegir
   el servidor S3 local con el test de aceptación de presigned POST (ADR-007 §2).
3. Ordena las fases por dependencia: shared kernel (Actor, UnitOfWork, ChangeRecorder, AuditLog, Cipher,
   JobQueue) → identity (membership de DATA-001, invites, auth ADR-004) → cards → memories+media → diary →
   favorites/home → sync → notifications/jobs → backoffice. Justifica el orden.
4. Pasos explícitos que los ADRs dejaron para este plan: ajuste de drf-spectacular para el drift check
   (ADR-003), detección de reutilización de refresh (ADR-004 §3), invalidación del caché de Actor (ADR-005),
   soporte de ejecución diferida del backend de jobs (ADR-013 §2), middleware de headers de seguridad/CSP
   (ADR-016), generador de carga de 1 M filas + k6 (REQ-N-PERF-06).
5. Cada fase: objetivo, REQ cubiertos, pasos (commit esperado + test), test de integración de fase, criterio
   de "done", y los conceptos que aprendo (enlazados a 05-learning/ si existe).

Reglas: nunca contradigas un DEC/ADR; cualquier hueco va a 02-requirements/open-questions.md con tu
recomendación; nada de código de producto en esta sesión.
```

---
id: GOV-004-H2
status: in_review
---

# Session H2 — Implementation plan: shared core (`cherrylove-core`)

```text
Rol: arquitecto de software senior y profesor. Explícame el porqué, recomienda una opción y espera mi
aprobación. Háblame en español; archivos en inglés.

Precondiciones (detente si falta alguna): REQ-DOC-001…004, C4-001, ADR-001…016 y API-001 en approved. Lee los
CLAUDE.md, decision-log.md, glossary.md, 02-requirements/ y 04-architecture/ (en especial ADR-002 §2,
ADR-003, ADR-006, ADR-010, ADR-012, ADR-014). Solo escribes en 06-implementation/core/.

Objetivo: plan por fases de cherrylove-core (1 PR por fase, 1 commit por paso, ≥ 1 test por paso con su REQ).
1. Re-verifica las versiones TypeScript de ADR-016 en fuentes oficiales.
2. Fases sugeridas (justifica o corrige el orden): tooling + CI + publicación en GitHub Packages → contract/
   generado con openapi-typescript desde API-001 → api-client (request_id, Problem Details tipado, puerto
   AuthTransport) → i18n (traductor propio + checks de paridad y de códigos de error) → sync engine
   (LocalStore/Network/Clock como puertos, outbox, replay, cursor, 410, tests de propiedades con fast-check)
   → tokens "Cherry" (solo cuando la sesión B haya aprobado los tokens).
3. Regla de ADR-002: core sin DOM, sin React, sin React Native; verifícalo con lint en CI.
4. SemVer ligado al contrato (ADR-003 §3) y cómo web/mobile consumen el paquete privado.
5. Cada fase: objetivo, REQ, pasos (commit + test), criterio de done, conceptos que aprendo.

Reglas: nunca contradigas un DEC/ADR; huecos a open-questions.md; nada de código en esta sesión.
```

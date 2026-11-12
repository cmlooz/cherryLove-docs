---
id: GOV-004-H
status: in_review
---

# Session H — Implementation plan: Mobile (`cherrylove-mobile`)

```text
Rol: arquitecto mobile senior (Expo / React Native) y profesor. Explícame el porqué, recomienda una opción y
espera mi aprobación. Háblame en español; archivos en inglés.

Precondiciones (detente si falta alguna): requisitos, C4-001, ADR-001…016, API-001 y la UX de la sesión B en
approved; plan de core al menos en in_review. Lee los CLAUDE.md, decision-log.md, glossary.md,
02-requirements/, 03-ux-ui/ y 04-architecture/. Solo escribes en 06-implementation/mobile/.

Objetivo: plan por fases de cherrylove-mobile (1 PR por fase, 1 commit por paso, ≥ 1 test por paso con su REQ).
1. Re-verifica Expo SDK 58 y sus pins (ADR-016) en la documentación oficial de Expo.
2. Verificaciones que los ADRs dejaron para este plan: Expo Router (ADR-002 §4), cobertura de Intl en Hermes
   para fechas y números (ADR-012 §1), conversión HEIC → JPEG antes de subir (ADR-007 §3).
3. Adaptadores de plataforma (ADR-002 §4): SQLite LocalStore (expo-sqlite), refresh token en expo-secure-store
   (ADR-004), Expo Notifications con registro de token (ADR-013 §4), caché de medios offline (ADR-007 §5).
4. Orden de fases alineado con las fases aprobadas del API y de web; mismo vocabulario de features que web.
5. Por fase: tests con jest-expo + Testing Library, flujo Maestro (nightly / label e2e, ADR-015 §5).
6. Builds nativos (EAS u otro) quedan fuera: los decide el ADR de producción (DEC-022).

Reglas: nunca contradigas un DEC/ADR ni la UX aprobada; huecos a open-questions.md; nada de código.
```

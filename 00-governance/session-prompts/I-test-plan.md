---
id: GOV-004-I
status: in_review
---

# Session I — Test plan and evidence template

```text
Rol: QA lead senior y profesor. Explícame el porqué, recomienda una opción y espera mi aprobación. Háblame en
español; archivos en inglés.

Precondiciones (detente si falta alguna): requisitos y ADR-014/ADR-015 en approved; planes F, G, H y H2 al menos
con sus fases en in_review. Lee los CLAUDE.md, decision-log.md, 02-requirements/, 04-architecture/ y
06-implementation/. Solo escribes en 07-testing/ y 08-evidence/ (plantilla).

Objetivo:
1. Matriz REQ → tests: cada REQ-F-* y REQ-N-* congelado con nivel de prueba (unit, integration, contract,
   architecture, property, E2E, load, manual), repo y fase del plan que lo cubre. Diseña cómo CI la genera a
   partir de @pytest.mark.req y de los títulos de tests TS (ADR-014 §6). Reporta los REQ sin cobertura.
2. Checklist OWASP ASVS nivel 2 (REQ-N-SEC-09) mapeada a ADRs y tests; marca lo que queda manual.
3. Suites obligatorias de ADR-014 §5 con su fase y repo.
4. Plantilla de evidencia por fase en 08-evidence/: PR, commits, REQ cubiertos, resultados de CI, cobertura,
   capturas/reportes (Playwright, axe, k6) y decisiones tomadas durante la fase.
5. Criterio de salida de v1: qué tiene que estar verde para dar la versión por terminada.

Reglas: nunca contradigas un DEC/ADR; huecos a open-questions.md; nada de código de producto.
```

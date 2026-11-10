---
id: GOV-004-C
status: in_review
---

# Session C — Architecture

```text
Rol: arquitecto de software senior y profesor. Explícame el porqué de cada decisión, compara alternativas
y recomienda una. Háblame en español; archivos en inglés.

Contexto: lee ../CLAUDE.md, cherryLove-docs/CLAUDE.md, STATUS.md (verifica REQ-DOC-001…004 approved;
si no, detente), decision-log.md, glossary.md y 02-requirements/. Solo escribes en 04-architecture/.

Objetivo (cada ADR se aprueba por separado; formato MADR: context, options, decision, consequences, REQ IDs):
1. C4 nivel 1 y 2 (contexto y contenedores) de los 5 repos y servicios externos.
2. ADRs, en este orden: monolito modular hexagonal y reglas de dependencia (DEC-002); estructura de carpetas
   por repo con ejemplos de dónde va cada responsabilidad (controller/view, use case, domain, repository,
   adapter, middleware, serializer, hook, service); contrato OpenAPI contract-first y versionado; auth
   (JWT, rotación, cookies web, secure storage móvil); autorización por pareja; modelo de sync offline
   (UUIDv7, client_op_id, version, tombstones, cursor); almacenamiento de medios (S3/MinIO, URLs firmadas,
   cuotas); auditoría y particionado de logs; cifrado en reposo; manejo de errores RFC 9457; caché;
   i18n; notificaciones; estrategia de pruebas; CI.
3. Modelo de datos (ERD) con índices y restricciones que hagan cumplir REQ-F-AUTH-05 en base de datos.
4. Esqueleto OpenAPI v1 con los endpoints de cada módulo.
5. Al final, verifica las versiones estables actuales de Python, Django, DRF, PostgreSQL, Node, React, Expo
   y TypeScript en la documentación oficial y propón versiones exactas fijadas en un ADR.
6. Redacta los prompts de las sesiones F, G, H, H2 e I en 00-governance/session-prompts/.

Reglas: cada librería nueva necesita justificación (pocas dependencias); nunca contradigas un DEC;
pregunta ante cualquier hueco; nada de código de producto.
```

---
status: in_review
---

# Architecture

Owned by session C. Every record here is approved one by one; nothing below `approved` may drive code.
Format: MADR (context → options → decision → consequences → REQ IDs).

## Index

| ID | Title | File | Status |
|----|-------|------|--------|
| C4-001 | C4 level 1 and 2 (context, containers) | `c4-context-containers.md` | **approved** |
| ADR-001 | Modular monolith, hexagonal layers, dependency rules | `adr-001-modular-monolith.md` | **approved** |
| ADR-002 | Folder structure per repo | `adr-002-folder-structure.md` | **approved** |
| ADR-003 | OpenAPI contract-first and versioning | `adr-003-openapi-contract.md` | **approved** |
| ADR-004 | Authentication (JWT, rotation, web cookies, mobile secure storage) | `adr-004-authentication.md` | **approved** |
| ADR-005 | Couple-scoped authorization | `adr-005-couple-authorization.md` | **approved** |
| ADR-006 | Offline sync model (UUIDv7, client_op_id, version, tombstones, cursor) | `adr-006-offline-sync.md` | **approved** |
| ADR-007 | Media storage (S3/MinIO, signed URLs, quotas) | `adr-007-media-storage.md` | **approved** |
| ADR-008 | Audit log and partitioning | `adr-008-audit-log.md` | **approved** |
| ADR-009 | Encryption at rest | `adr-009-encryption-at-rest.md` | **approved** |
| ADR-010 | Error handling (RFC 9457) | — | not_started |
| ADR-011 | Caching | — | not_started |
| ADR-012 | i18n | — | not_started |
| ADR-013 | Notifications and background jobs | — | not_started |
| ADR-014 | Testing strategy | — | not_started |
| ADR-015 | CI | — | not_started |
| DATA-001 | Data model (ERD, indexes, constraints) | — | not_started |
| API-001 | OpenAPI v1 skeleton | — | not_started |
| ADR-016 | Pinned versions | — | not_started |

Order follows `00-governance/session-prompts/C-architecture.md`. Numbers are reserved; a rejected ADR keeps
its number with `status: rejected`.

---
id: GOV-003
status: approved
approved_by: Camilo
approved_at: 2026-11-09
---

# Decision log

Append-only. Each row was answered explicitly by Camilo. Changing a decision = new row that supersedes the old one.

## 2026-11-09 — Session 0/A (requirements)

| ID | Topic | Decision |
|----|-------|----------|
| DEC-001 | Prior "couple-app" project | Start from scratch; new repos. couple-app is not reused. |
| DEC-002 | Backend architecture | Modular monolith, pragmatic hexagonal (plain-Python domain, ORM behind repositories). |
| DEC-003 | Web vs mobile | Separate UIs (React DOM / React Native) + shared core package. |
| DEC-004 | Repos | Private polyrepo: `cherrylove-api`, `cherrylove-web`, `cherrylove-mobile`, `cherrylove-core` (GitHub Packages), `cherryLove-docs`. |
| DEC-005 | Admin app | Customized Django Admin + custom views. |
| DEC-006 | Several cards same day | Carousel, same order for both. |
| DEC-007 | Future cards | Partner sees a counter only. |
| DEC-008 | Album auto-scroll | Photo by photo per date; pause on touch; resume after ~8 s. |
| DEC-009 | Uploaded files | S3-compatible object storage (MinIO locally) + device cache. |
| DEC-010 | Music | Mixed: file/URL = background audio; Spotify/YouTube = visible embed. |
| DEC-011 | Favorites | Per user; home timeline shows both partners' with who-starred indicator. |
| DEC-012 | Place | First-class entity; 1 place → N memories. |
| DEC-013 | Maps | OpenStreetMap. |
| DEC-014 | Diary visibility | `shared` (default) or `only_me`. |
| DEC-015 | Staff privacy | Metadata only, never content. |
| DEC-016 | Staff roles | superadmin, admin, support (read-only). |
| DEC-017 | Invite expiry | 72 h, resendable. |
| DEC-018 | Partial onboarding | One partner alone can already use the app. |
| DEC-019 | Couple ends | Archive → 90 days → purge. |
| DEC-020 | Quotas | Configurable; defaults 5 GB/couple, 20/200/30 MB photo/video/audio. |
| DEC-021 | UI language | Spanish and English from v1. |
| DEC-022 | Mobile & local env | Expo + Docker Compose; production in a later ADR. |
| DEC-023 | Tests & commits | ≥ 1 test per code step + integration per phase; Conventional Commits; 1 commit/step; 1 PR/phase; coverage services ≥ 90 %, overall ≥ 80 %. |
| DEC-024 | Card lock & timezone | Card locked once published; "today" in couple timezone. |
| DEC-025 | Diary & account | Star on notes; name editable; birth date admin-only. |
| DEC-026 | v1 extras | Password recovery, push, anniversary/birthday reminders. |
| DEC-027 | Backlog | Export book, dark mode, OpenTelemetry, load generator (see OQ-001), feature flags. |
| DEC-028 | Who creates a couple | The admin, with two emails. No self-service. |
| DEC-029 | Django API style | Django REST Framework. |
| DEC-030 | Docs language | English. Spanish v0 drafts from session 0 are superseded by this repo. |
| DEC-031 | Conflicting recommendations | Today's approved decisions prevail over unanswered v0 recommendations. |

## 2026-11-10 — Workspace preparation

| ID | Topic | Decision |
|----|-------|----------|
| DEC-032 | Git identity | `cmlooz <84087705+cmlooz@users.noreply.github.com>` (GitHub noreply). |
| DEC-033 | Code repos timing | Code repos are created after the requirements freeze and architecture ADRs (session R). |

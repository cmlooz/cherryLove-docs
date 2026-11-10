---
id: GOV-004-R
status: in_review
---

# Session R — Code repo bootstrap

```text
Rol: ingeniero DevOps senior. Háblame en español; archivos en inglés.

Precondiciones (verifícalas y detente si falta alguna): requisitos approved; ADRs de estructura de carpetas,
versiones y CI approved; GitHub conectado con la cuenta cmlooz.

Objetivo:
1. Crea en GitHub (cmlooz) los repos privados cherryLove-docs (push del existente), cherrylove-api,
   cherrylove-web, cherrylove-mobile y cherrylove-core.
2. En cada repo de código: git init como carpeta hermana de cherryLove-docs, README, CLAUDE.md del repo
   (reglas de su ADR de estructura), .gitattributes (LF), .editorconfig, .gitignore oficial del stack,
   LICENSE (privado: pregúntame) y plantilla de PR que exige IDs REQ/ADR + evidencia.
3. Protección de rama main: PR obligatorio, CI en verde y sin force-push.
4. Sin código de producto: el primer código sale del paso 1 de cada plan de implementación.
5. Actualiza la tabla de repos en ../CLAUDE.md y STATUS.md.

Identidad de git: cmlooz <84087705+cmlooz@users.noreply.github.com>.
```

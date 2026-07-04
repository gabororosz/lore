# E04 — Auth (GitHub OAuth) + adatmodell

**Forrás:** koncepció (Jogosultság és szerzőség; GitHub OAuth + GitHub App modell), technikai terv §2 (adatmodell).
**Cél:** GitHub OAuth belépés, stabil emberi identitás a `--author`-hoz, teljes adatmodell (a tárolási filozófia három kategóriája szerint), tenant-izoláció.
**Függőség:** E01. (A GitHub App installation-oldala E05.)
**Tesztek:** AUTH-01, AUTH-02, AUTH-03.

## Döntések (a tervből rögzítve)

- **GitHub OAuth az ember azonosítására** → stabil `gh_user_id`, login, név, commit-authorhoz használható e-mail.
- A felhasználónak **nincs szüksége write-jogra**; a push a boté. A hozzáférési kapu: **explicit repo-hozzáférés** (privát/internal: repo read permission; publikus: collaborator/org membership vagy Lore-meghívás; puszta public read **nem** ad szerkesztési jogot).
- Jogosultság-ellenőrzés **cache-sel, rövid TTL-lel**; kritikus műveletnél (repo megnyitás, Publish, PR) újraellenőrzés.
- **E-mail fallback:** ha nincs commit-authorhoz használható e-mail → GitHub noreply cím (`<id>+<login>@users.noreply.github.com`).
- A Git repo **all-or-nothing** hozzáférésű — nincs per-fájl jogosultság (dokumentált korlát).

## Taskok — Auth

- [ ] **E04-T1** GitHub OAuth flow (login/callback), session-kezelés. Kimenet: stabil `gh_user_id`, `login`, `name`, `email`.
- [ ] **E04-T2** E-mail-feloldás a `--author`-hoz + noreply fallback. **AUTH-02** (email elérhető a commithoz).
- [ ] **E04-T3** Jogosultság-ellenőrző szolgáltatás: explicit repo-hozzáférés vizsgálata GitHub API-val, cache rövid TTL-lel, re-validáció kritikus műveletnél.
- [ ] **E04-T4** **Tenant-izoláció** minden lekérdezésben (row-level scoping): user csak a saját tenant repóit éri el. **AUTH-03**.
- [ ] **E04-T5** Publikus repo read-only mód (opcionális termékdöntés): anonim/public-read user nem kap draftot, Editet, Publisht, PR-t.

## Taskok — Adatmodell (Postgres séma + migrációk)

A modell a tárolási filozófia három kategóriája mentén tagolt.

**Valódi app-állapot (backup-olandó):**
- [ ] **E04-T6** `tenant` — `id`, `gh_installation_id`, `retention_config`.
- [ ] **E04-T7** `user` — `gh_user_id`, `login`, `email`, `name`.
- [ ] **E04-T8** `repository` — `tenant_id`, `gh_repo_id`, `default_branch`, `docs_path` (default `/docs`), `docs_sync_branch`.
- [ ] **E04-T9** `audit_event` — **tartalom-mentes evidencia**: `user_id`, `repo_id`, `doc_id`, `action`, `commit_sha`, `created_at` (a GDPR↔AI Act feloldás: metaadat, nem szöveg).
- [ ] **E04-T10** `pull_request` — `repo_id`, `gh_pr_number`, `state` (a badge-hez).

**Derivált (eldobható, a mirrorból újraépíthető) — sémát itt, feltöltést E07:**
- [ ] **E04-T11** `document` — `repo_id`, `doc_id`, `current_path`, `title`, `owner`, `status`, `tags`, `docs_sync_sha`.
- [ ] **E04-T12** `doc_link` — `from_doc_id`, `to_doc_id`.
- [ ] **E04-T13** `doc_issue` — `repo_id`, `doc_id`/`path`, `issue_type` (missing_id/duplicate_id/invalid_frontmatter/broken_link), `detail`, `detected_at`.
- [ ] **E04-T14** Postgres FTS keresési index oszlop/tábla (feltöltés E07, lekérdezés E14).

**Átmeneti (szerver-tulajdon, TTL-es) — sémát itt, logikát E09/E10:**
- [ ] **E04-T15** `draft` — `user_id`, `repo_id`, `doc_id` vagy `target_path`, `base_sha`, `content`, `created_at`, `updated_at`, `state` (active/dormant), `expires_at`.
- [ ] **E04-T16** `lock` — `doc_id`, `holder_user_id`, `acquired_at`, `expires_at` (elsődlegesen Redis; Postgres csak ha durabilitás kell).
- [ ] **E04-T17** Migrációs eszköz beállítása (pl. node-pg-migrate/Prisma migrate) + seed dev-adat.

## Kész-definíció

- GitHub-bejelentkezés után a user profilja (login+email) elérhető.
- A séma migrálható tiszta DB-re; a tenant-izoláció teszttel igazolt (AUTH-03).
- Jogosulatlan user nem ér el idegen tenant-repót.

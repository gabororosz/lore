# E01 — Projekt-alapok, monorepo, dev-környezet

**Forrás:** technikai terv §1 (technológiai stack), átfogó elv (cloud-agnoszticitás).
**Cél:** működő monorepo közös típusokkal, futtatható üres frontend + backend, helyi fejlesztői környezet.
**Függőség:** nincs (belépő epic).

## Döntések (a tervből rögzítve)

- Frontend: **TypeScript + React**, PWA.
- Backend: **TypeScript/Node — Fastify** (nem NestJS: solo fejlesztő, vékony framework illik az SSE + git-subprocess mintákhoz).
- Közös típusok a frontend és backend között (monorepo előny).
- Minden réteg **cserélhető interfész** mögött (git-réteg, transport, LLM, objektumtár) — standard building blockok, nincs lock-in primitív.

## Taskok

- [ ] **E01-T1** Monorepo felállítása (pnpm/npm workspaces vagy Turborepo). Csomagok: `apps/web` (React PWA), `apps/api` (Fastify), `packages/shared` (közös TS-típusok, domain-modellek).
- [ ] **E01-T2** TypeScript strict konfiguráció, közös `tsconfig` base; ESLint + Prettier egységesen a workspace-en.
- [ ] **E01-T3** `packages/shared`: az adatmodell és API-kontraktus típusai (tenant, user, repository, document, draft, lock, doc_issue, audit_event, pull_request — lásd E04). Ez a frontend↔backend szerződés forrása.
- [ ] **E01-T4** Fastify csontváz: healthcheck endpoint, konfig-betöltés (env), strukturált (tartalom-mentes) logging alap (a compliance-elv szerint: minimalizált, tartalom-mentes napló).
- [ ] **E01-T5** React PWA csontváz: Vite + React, PWA-manifest, service worker váz (offline piszkozat-cache később, E15).
- [ ] **E01-T6** Cserélhető interfészek deklarálása `packages/shared`-ben (portok): `GitPort`, `TransportPort` (presence/lock-csatorna), `LlmPort`, `BlobStorePort`. Implementáció a saját epicjeikben.
- [ ] **E01-T7** Helyi dev-környezet: `docker-compose` Postgres + Redis + (GCS-emulátor vagy MinIO S3-kompatibilisen). Egyparancsos `dev` script (api + web + infra).
- [ ] **E01-T8** `.env.example` + titkos konfig kezelése dev-ben (Secret Manager prod-ban, E02); dokumentált setup a repo README-ben.
- [ ] **E01-T9** Alap monorepo scriptek: `lint`, `typecheck`, `test`, `build`, `dev` — a CI (E03) ezekre épül.

## Kész-definíció

- `npm run dev` felhozza az apit + webet + a helyi Postgres/Redis/objektumtárat.
- A frontend hívja a backend healthcheckjét közös típusokkal.
- Lint + typecheck + üres teszt zölden fut.

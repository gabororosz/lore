# Lore — Task-backlog

Ez a mappa a `../docs/` tervcsomag (koncepció, technikai terv, UX-terv, teszteset-katalógus, tárolási filozófia) végrehajtható task-bontása. A cél az MVP leszállítása.

## Hogyan olvasd

- A taskok **epicekbe** (`E01`…`E17`) vannak rendezve, mindegyik külön fájlban.
- Minden task azonosított: `E<epic>-T<n>` (pl. `E06-T3`).
- Minden epic hivatkozik a forrás-dokumentum szakaszára és a releváns tesztesetekre (`SER-*`, `MRG-*`, stb. — lásd `../docs/lore-teszt-esetek.md`).
- A checkboxok (`- [ ]`) a haladás követésére valók.

## Scope-elv

Csak az **MVP** van itt taskosítva. A dokumentumokban `post-MVP`-ként jelölt elemek (kód-tudatos AI, CRDT/valós idejű együttszerkesztés, brown-field migráció, `docs-sync` lifecycle-motor, vezetett invariáns-javítás „Réteg 3") **nincsenek** a backlogban — a `E17` végén külön „Post-MVP parkoló" szakasz emlékeztet rájuk.

## Epicek és függőségi sorrend

```
E01 Projekt-alapok ─┬─> E02 Infrastruktúra ──> E03 CI/CD
                    │
                    └─> E04 Auth + adatmodell ─┬─> E05 GitHub App + webhook ──> E06 Git-mirror ─┬─> E07 Index + invariáns
                                               │                                                 │
                                               │                                    E08 MD-szerializáció ─┐
                                               │                                                          ├─> E11 Publish/merge ──> E12 Propose to main
                                               ├─> E09 Piszkozat + retenció ─────────────────────────────┤
                                               │                                                          │
                                               └─> E10 Lock/presence + SSE ───────────────┬──────────────┘
                                                                                          └─> E13 AI (SSE-re épül)

E14 Frontend shell ──> E15 Editor ──> E16 Read-nézet + badge     (a backendre épülnek, de párhuzamosan indíthatók mockokkal)

E17 Tesztelés & QA — keresztvágó, végig fut minden epicen
```

## Javasolt megvalósítási hullámok (solo fejlesztő)

1. **Hullám 1 — Váz:** E01, E02, E03, E04 (auth + üres app deploy-olva).
2. **Hullám 2 — Git-gerinc:** E05, E06, E07 (repo bekötés → mirror → index → invariáns-detektálás).
3. **Hullám 3 — Szerkesztési mag:** E08, E09, E10 (szerializáció, piszkozat, lock/SSE).
4. **Hullám 4 — Publish-hurok:** E11, E12 (3-way merge, konfliktus, PR).
5. **Hullám 5 — Kliens:** E14, E15, E16 (a teljes UX; a backendre kötve).
6. **Hullám 6 — AI:** E13.
7. **Végig:** E17 (a tesztek epicről epicre születnek, nem a végén).

## Load-bearing kockázatok (kiemelt figyelem)

A technikai terv három magas kockázatú területet nevez meg — ezekre külön coverage-kapu:

- **Publish 3-way merge** (`E11`) — adatvesztés-veszély.
- **Markdown szerializáció round-trip** (`E08`) — a „reviewzható, mint a kód" ígéret.
- **Frontmatter-invariánsok** (`E07`) — `id`-megőrzés, komment túlélése, wiki-link-feloldás.

## Epic-lista

| Epic | Téma | Fő forrás |
|------|------|-----------|
| [E01](01-projekt-alapok.md) | Projekt-alapok, monorepo, dev-környezet | technikai terv §1 |
| [E02](02-infrastruktura-iac.md) | Infrastruktúra & IaC (GCP, Terraform) | technikai terv §3 |
| [E03](03-cicd.md) | CI/CD pipeline | technikai terv §4 |
| [E04](04-auth-adatmodell.md) | Auth (GitHub OAuth) + adatmodell | koncepció (jogosultság), technikai terv §2 |
| [E05](05-github-app-webhook.md) | GitHub App + webhook | koncepció (architektúra), technikai terv §1 |
| [E06](06-git-mirror.md) | Git-integrációs réteg (sparse mirror) | koncepció (mirror), technikai terv §1 |
| [E07](07-index-invarians.md) | Derivált index + invariáns-validátor | koncepció (wiki-link), technikai terv §6 |
| [E08](08-markdown-szerializacio.md) | Markdown szerializáció (minimal-diff) | koncepció (szerkesztő), technikai terv §5 |
| [E09](09-piszkozat-retencio.md) | Piszkozat-tár + retenció | koncepció (retenció), technikai terv §2 |
| [E10](10-lock-presence-sse.md) | Lock/presence + SSE transport | koncepció (konkurencia, realtime) |
| [E11](11-publish-merge.md) | Publish 3-way merge + konfliktus | koncepció (publish), technikai terv §5 |
| [E12](12-propose-to-main.md) | Propose to main (PR) | koncepció (Git integráció) |
| [E13](13-ai-szolgaltatas.md) | AI-szolgáltatás (provider-absztrakció + streaming) | koncepció (AI) |
| [E14](14-frontend-shell.md) | Frontend app shell + PWA + navigáció | UX-terv (IA, felületek 1–3, 9–11) |
| [E15](15-editor.md) | WYSIWYG szerkesztő (TipTap) | UX-terv (felület 4, 6), technikai terv §5 |
| [E16](16-read-badge-degradation.md) | Read-nézet + badge + graceful degradation UI | UX-terv (felület 5, badge, DEG) |
| [E17](17-teszteles-qa.md) | Tesztelés & minőségbiztosítás | technikai terv §5, teszteset-katalógus |

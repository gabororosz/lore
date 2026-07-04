# E17 — Tesztelés & minőségbiztosítás (keresztvágó)

**Forrás:** technikai terv §5 (tesztelési stratégia), teljes teszteset-katalógus (`../docs/lore-teszt-esetek.md`).
**Cél:** kockázat-alapú teszt-piramis és teszt-infrastruktúra, amely végig fut minden epicen — nem a végén, hanem epicről epicre.
**Függőség:** keresztvágó (minden epic tesztjei ide is tartoznak).

## Vezérelv (a tervből rögzítve)

A tesztelési erőfeszítés oda összpontosul, ahol egy hiba **adatvesztést/-korrupciót** okoz. Három load-bearing kockázat:
1. **Publish 3-way merge** (E11) — MRG-*
2. **Markdown szerializáció round-trip** (E08) — SER-*
3. **Frontmatter-invariánsok** (E07) — FM-*

## Taskok — Teszt-infrastruktúra

- [ ] **E17-T1** **testcontainers** Postgres + Redis a CI integrációs tesztjeihez (valódi függőségek, nem mock).
- [ ] **E17-T2** **Teszt-GitHub-org + teszt-GitHub-App** (E03-T6) + **efemer teszt-repók** tesztfutásonként létrehozva/lebontva (a git-réteg valódi repókon dolgozik).
- [ ] **E17-T3** Webhook-fixtúrák (E05-T8) record/replay a kontraktus-tesztekhez.
- [ ] **E17-T4** **LLM-stub** a provider-absztrakció mögött (E13-T7): a CI nem hív valódi LLM-et; valódi LLM csak külön, ritka, jelölt tesztben.
- [ ] **E17-T5** **Playwright** mobil-viewporttal + touch-emulációval (mobile-first e2e).

## Taskok — Teszt-piramis (rétegenként, az ID-k a katalógusból)

- [ ] **E17-T6** **Unit (a legtöbb teszt):** szerializáció golden-file (SER-01…11), frontmatter (FM-01…09), wiki-link-feloldás (LINK-01/04/06), lock-állapotgép, retenciós állapotgép (DRAFT-05).
- [ ] **E17-T7** **Property/fuzz:** round-trip stabilitás (SER-12/13/14), merge-biztonság (MRG-10/11).
- [ ] **E17-T8** **Integráció (valódi függőségek):** git-réteg valódi git CLI ellen efemer repókon (MRG-01…08, GH-04/05/08/09, LINK-02/03/05); DB+Redis testcontainers (LOCK-*, DRAFT-*, SSE-01…05, AUTH-*, IMG-02/03).
- [ ] **E17-T9** **Kontraktus/komponens:** GitHub App webhook (GH-01/02/03/06/07), aláírás-ellenőrzés.
- [ ] **E17-T10** **E2E (fő hurok):** megnyitás → Edit (lock) → szerkesztés → Publish → badge (MRG-09, READ-02…05, DEG-01/03/04, IMG-01, GH-10, MOB-04).
- [ ] **E17-T11** **Terhelés:** párhuzamos SSE-session kapacitás (SSE-07, kötődik E02-T9/E10-T15).
- [ ] **E17-T12** **Manuális mobil-kör:** IME/érintőbevitel iOS Safari + Android Chrome (MOB-01/02/03) — kiadás előtt.

## Taskok — CI-integráció (E03-ra épül)

- [ ] **E17-T13** Minden PR-en: unit + property + testcontainers-integráció (gyors, izolált).
- [ ] **E17-T14** Pre-deploy/ütemezett: kontraktus + sandbox-e2e (teszt-org) + SSE-07 terhelés.
- [ ] **E17-T15** **Coverage-kapu** a kritikus magra: SER, FM, MRG modulok (nem globális százalék).
- [ ] **E17-T16** Amit tudatosan **NEM** tesztelünk túl: a derivált index *rebuild* helyességét igen, de nem minden lekérdezést; a presence flakiness alacsony tét (nincs E2E-súly).

## Kész-definíció

- A teszt-piramis minden szintje fut a CI-ben; a load-bearing modulok coverage-kapun vannak.
- A katalógus minden **K (kritikus)** esete le van fedve; az e2e fő hurok a teszt-orgon zöld.

---

## Post-MVP parkoló (NEM az MVP-backlog része — emlékeztető)

A dokumentumok tudatosan post-MVP-re halasztott elemei, hogy ne vesszenek el:

- **Kód-tudatos AI** — a teljes repo (`src/`, `docs/`, `.github/`, `terraform/`) együttes értelmezése; retrieval/embedding réteg; ehhez tartozó adatkormányzás (per-repo opt-in). *A hosszú távú moat.*
- **CRDT / valós idejű együttszerkesztés** — a cserélhető lock-policy és transport-absztrakció (E10) készen áll a melléépítésre.
- **Brown-field migráció** — Confluence/Jira/Slack → repo. Az egyetlen tudatosan nyitva hagyott koncepcionális kérdés.
- **`docs-sync` lifecycle-motor** — visszaszinkron a `main`-ről, elutasított/módosított PR-ek, közvetlen `main`-változás, branch-protection, force push, branch-újraalapozás.
- **Invariáns Réteg 3 — vezetett javítás** — egykattintásos, attribútált committal menő javítási javaslatok a `doc_issue`-khoz.
- **Git LFS** — csak ha a valós kép-volumen a feltételezett nagyságrend fölé megy.
- **EU-szuverén hosting-migráció** — Scaleway/Hetzner/OVHcloud, ha egy insurance-ügyfél a valódi szuverenitást megköveteli (a standard building blockok ezt lehetővé teszik).

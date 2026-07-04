# E07 — Derivált index + invariáns-validátor

**Forrás:** koncepció (Dokumentum-metaadatok, Wiki-linkek, Nyitott kérdések: invariáns-robusztusság), technikai terv §2, §6.
**Cél:** a mirrorból újraépíthető derivált index (id→fájl, wiki-link-gráf, frontmatter, keresés) + az invariáns-validátor (detektálás + graceful degradation).
**Függőség:** E06 (mirror + diff + commit-gráf), E04 (index-táblák sémája).
**Tesztek:** FM-01…09, LINK-01…06, DEG-01…04, GH-09 (duplikátum git-history).

## Döntések (a tervből rögzítve)

- A dokumentum-identitás a frontmatter **`id`** (nem útvonal, nem cím). Formátum: **slug + rövid random suffix** (pl. `payment-api-a3f9`).
- Wiki-link `[[id|Label]]` — **id alapján** oldódik fel; a label csak megjelenítés.
- **Invariáns-stratégia (MVP):** Réteg 1 detektálás + Réteg 2 graceful degradation + Réteg 4 megelőzés a Lore saját írásainál. **Réteg 3 (vezetett javítás) = post-MVP** (nincs itt).
- **Duplikátum-feloldás: a régebbi commit nyer** — git-history-lekérdezéssel (nem pillanatnyi fájlállapot).
- Az index derivált: a `doc_issue` is a syncnél újraépül; egy hibás doksi **nem** állítja le a rebuildet.

## Taskok — Indexelés

- [ ] **E07-T1** Frontmatter-parser: `id`, `title`, `owner`, `status`, `tags` kiolvasása. **FM-03**.
- [ ] **E07-T2** `document` index feltöltése/frissítése a mirror-diffből (id→`current_path`, title, owner, status, tags, `docs_sync_sha`).
- [ ] **E07-T3** Wiki-link-parser + `doc_link` gráfél építése/frissítése. **LINK-06**.
- [ ] **E07-T4** Postgres FTS index feltöltése (cím + tartalom) a kereséshez (lekérdezés E14).
- [ ] **E07-T5** Inkrementális index-rebuild a webhook-diffre (a teljes újraépítés csak reszinkronnál, E06-T5).

## Taskok — Wiki-link-feloldás

- [ ] **E07-T6** `[[id|Label]]` feloldás **id alapján**; a feloldás túléli az átnevezést és áthelyezést. **LINK-01, LINK-02, LINK-03**.
- [ ] **E07-T7** Duplikált cél-id feloldása: a **régebbi commit** doksijára oldódik — **git-history-lekérdezéssel** (E06-T7). **LINK-05, GH-09**.
- [ ] **E07-T8** Törött link (nem létező id) → `doc_issue: broken_link` + „törött" render-jelzés (UI E16). **LINK-04**.

## Taskok — Invariáns-validátor (Réteg 1 + 2)

- [ ] **E07-T9** Validátor a mirror-sync részeként: hiányzó id → `missing_id` (**FM-05**); érvénytelen YAML → `invalid_frontmatter`, **nincs crash** (**FM-04**); sérült `---` / hiányzó frontmatter → kezelt jelzés (**FM-06, FM-07**).
- [ ] **E07-T10** Duplikált id detektálás → `duplicate_id` az újabbon (a régebbi nyer). **LINK-05**.
- [ ] **E07-T11** **Graceful degradation:** egy hibás doksi nem állítja le az index-rebuildet; a többi doksi rendben feldolgozódik. **DEG-02**.
- [ ] **E07-T12** `doc_issue` rekordok írása minden talált problémára (a UI ezekből jelez, E16). Nem állítja le a rendszert.

## Taskok — Réteg 4 (megelőzés a Lore saját írásainál)

- [ ] **E07-T13** **Egyedi id-generátor** (slug + random suffix) a Lore saját dokumentum-létrehozásánál. **FM-08**. (Suffix hossza/ábécéje iterálható.)
- [ ] **E07-T14** Publish előtti frontmatter-validáció a Lore saját írásánál. **FM-09**. (Kötődik E11-hez.)

## Kész-definíció

- A mirror-sync után az index (document, doc_link, FTS) konzisztens; a wiki-linkek id-alapon feloldódnak és túlélik az átnevezést.
- Hibás doksi esetén `doc_issue` keletkezik, a rebuild nem áll le (DEG-02).
- Duplikált id-nél a régebbi commit nyer, git-history alapján (GH-09).
- A mirror eldobás+újraépítés azonos index-állapotot ad (kötődik GH-08).

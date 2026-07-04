# E13 — AI-szolgáltatás (provider-absztrakció + streaming)

**Forrás:** koncepció (AI-kontextus; AI: futási mód és LLM-elhelyezés; AI funkciók MVP), UX-terv (felület 6, AI-panel).
**Cél:** `/docs`-only, aszinkron + streamelő AI-szerkesztés a piszkozaton, provider-független interfész mögött, EU-rezidencia/zero-retention/no-training szerződéses minimummal.
**Függőség:** E09 (draft), E10 (SSE-csatorna), E01-T6 (`LlmPort`).
**Tesztek:** DRAFT-04, SSE-02, + LLM-stub (E17).

## Döntések (a tervből rögzítve)

- **Scope: kizárólag a `/docs`** — nem küldünk forráskódot LLM-nek. Nem chatbot: konkrét dokumentumszerkesztési feladatok (javítás, összefoglaló, átírás, utasítás-alapú módosítás) a piszkozaton.
- **Futási mód: aszinkron + streaming** — a kérés job-azonosítót ad, az eredmény a **meglévő SSE-csatornán** (E10) érkezik, tokenről tokenre.
- **LLM-elhelyezés: elv-alapú, provider-független** — `LlmPort` absztrakció; szerződéses minimum: EU-adatrezidencia + zero-retention + no-training, DPA-ban al-adatfeldolgozóként. **Konkrét szolgáltató nincs rögzítve.**
- Az LLM **stateless compute** — minden perzisztencia (piszkozat, audit) a Lore EU-rezidens rétegében.
- **Kód-tudatos AI (teljes repo) = post-MVP** (nincs itt).

## Taskok

- [ ] **E13-T1** `LlmPort` interfész + legalább egy konkrét adapter (EU-régiós kereskedelmi API a szerződéses minimummal). Provider kód-átírás nélkül cserélhető.
- [ ] **E13-T2** AI-orkesztráció: `/docs`-méretű doksi/piszkozat + utasítás → módosított Markdown. **Alkalmazási szintű `/docs`-korlát** (forráskód sosem megy LLM-hez).
- [ ] **E13-T3** Aszinkron job-modell: a kérés azonnal job-azonosítót ad (nem blokkoló).
- [ ] **E13-T4** **Streaming** az eredményről az SSE-csatornán a **draftba** (E10-T10). **SSE-02, DRAFT-04**.
- [ ] **E13-T5** Az AI-módosítás **jelölt** (mi változott); a felhasználó `Elfogad`/`Elvet`/`Finomít` (UI E15). Az AI sosem publikál közvetlenül — a Draft→Publish adja az emberi jóváhagyást.
- [ ] **E13-T6** Transzparencia + audit: az AI-módosításról jelzés; `audit_event` (tartalom-mentes) a jóváhagyáskor/publishkor.
- [ ] **E13-T7** **LLM-stub** a provider-absztrakció mögött a CI-hez: rögzített fixtúra/stub — a CI **nem** hív valódi LLM-et (költség + nemdeterminizmus). Valódi LLM csak külön, ritka, jelölt tesztben (E17).
- [ ] **E13-T8** Költség-/méret-védelem: a `/docs`-doksi mérete korlátozott az egy-lövéses feladathoz (unit economics).

## Kész-definíció

- Utasításra az AI a piszkozatot módosítja, az eredmény streamelve épül (SSE-02, DRAFT-04).
- A módosítás jelölt, elfogadható/elvethető; az AI nem publikál közvetlenül.
- A CI stubbal fut, valódi LLM-hívás nélkül; a provider cserélhető.
- Forráskód soha nem kerül az LLM-hez (alkalmazási `/docs`-korlát igazolva).

# E16 — Read-nézet + badge-rendszer + graceful degradation UI

**Forrás:** UX-terv (felület 5 read, Állapot-badge rendszer, Degradált/hibaállapotok), koncepció (Read-forrás, háromállapotú badge).
**Cél:** a read-first dokumentum-nézet, a `main`-hez viszonyított barátságos állapot-badge-ek, és a graceful degradation UI-viselkedése.
**Függőség:** E07 (index, wiki-link, doc_issue), E12 (PR-állapot), E10 (lock/presence), E14 (shell).
**Tesztek:** READ-01…05, IMG-01, DEG-01…04, LINK-04.

## Döntések (a tervből rögzítve)

- Az olvasónézet **mindig a `docs-sync`-ből** olvas (nem `main`) — hogy a saját publikálás azonnal látsszon. **READ-01, READ-02**.
- **Háromállapotú badge** a `main`-hez viszonyítva: Szinkronban (nincs badge) / Friss verzió (halk) / Review alatt (PR-link). A „friss verzió" a *gyakori* köztes állapot → szándékosan visszafogott.
- **Nyers Git soha** — konfliktus, SHA, branch-parancs nem jelenik meg.
- **Graceful degradation:** a hiba **láthatóvá tett**, de nem rejtett és nem crashel; a Lore **nem javít automatikusan** (vezetett javítás = post-MVP).

## Taskok — Read-nézet (felület 5)

- [ ] **E16-T1** Renderelt Markdown (címsorok, listák, táblázatok, kódblokkok, képek, idézetek, checklisták) a `docs-sync`-ből. **READ-01**. Dokumentum alapból **read-only** nyílik. **IMG-01**.
- [ ] **E16-T2** Feloldott **wiki-linkek** (label megjelenítés, kattintható); törött link „törött" (404-szerű) render, a környező szöveg **nem tűnik el**. **LINK-04, DEG-04**.
- [ ] **E16-T3** Dokumentum-fejléc: cím, `owner`, `status`, `tags`, utolsó módosítás (szerző + idő Git author alapján).
- [ ] **E16-T4** `Edit` gomb (lock kérése, E10; belépés a szerkesztőbe, E15) — kiemelt, thumb-zónában mobilon. Ha más szerkeszti: „X szerkeszti" + `Átvétel kérése` lejárt/elhagyott locknál.
- [ ] **E16-T5** Kontextusmenü `⋯`: Propose to main (E12), Verziótörténet (Git-történet barátságos nézete), Link másolása, Dokumentum-infó.
- [ ] **E16-T6** Verziótörténet-nézet: legalább az utolsó módosítások szerzővel (mélység iterálható).

## Taskok — Badge-rendszer

- [ ] **E16-T7** Badge-számítás a `main` vs. `docs-sync` vs. PR-állapotból: Szinkronban → **nincs** badge (**READ-03**); Publikálva, nincs PR → „friss verzió" (**READ-04**); PR nyitva → „review alatt / javasolt" + opcionális PR-link (**READ-05**).
- [ ] **E16-T8** Saját publikálás **azonnal** látszik a read-nézetben (nem kell PR-merge). **READ-02**.
- [ ] **E16-T9** Badge megjelenítés a fában (E14-T7) és a dokumentum-fejlécben, egységes visszafogott vizuális súllyal.

## Taskok — Graceful degradation UI

- [ ] **E16-T10** Érvénytelen frontmatter → nyers, olvasható tartalom „⚠ probléma" jelöléssel; nem crashel. **DEG-01**.
- [ ] **E16-T11** Duplikált id → **mindkét** doksi látszik, ütközés-jelöléssel; a wiki-linkek a régebbi commit doksijára oldódnak (az újabbon jelölés). **DEG-03**.
- [ ] **E16-T12** ⚠ probléma-ikon a fában és a fejlécben (a `doc_issue` alapján, E07) — mindig látszik, nem rejtett.
- [ ] **E16-T13** Lock elhagyott/lejárt → `Átvétel` felkínálása; a korábbi user piszkozatához nincs hozzáférés (E10-T5).

## Kész-definíció

- A read-nézet a `docs-sync`-ből olvas, a saját publikálás azonnal látszik (READ-01/02).
- A badge helyesen tükrözi a szinkron/friss/review állapotot (READ-03/04/05).
- Hibás doksi „⚠ probléma"-ként, de olvashatóan jelenik meg; törött link nem tünteti el a szöveget (DEG-01/04).
- Nyers Git-fogalom sehol nem szivárog a felszínre.

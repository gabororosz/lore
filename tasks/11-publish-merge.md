# E11 — Publish 3-way merge + konfliktus

**Forrás:** koncepció (Publish mechanika: bázis-SHA + 3-way merge; a szekvenciadiagram), technikai terv §5 (a legmagasabb kockázat).
**Cél:** a piszkozat közzététele a `docs-sync`-re bázis-SHA + 3-way merge-dzsel, `--author` committal; konfliktus esetén barátságos felbontó, sosem nyers marker.
**Függőség:** E06 (git-műveletek), E09 (draft + bázis-SHA + draft-kép), E10 (lock felszabadítás), E08 (valid MD).
**Tesztek:** MRG-01…11, LOCK-07, DRAFT-08, IMG-02. **Load-bearing kockázat — coverage-kapu.**

## Döntések (a tervből rögzítve)

- A Publish **nem** naiv felülírás: bázis-SHA + `fetch` + **3-way merge** (ős = bázis-SHA, miénk = piszkozat, övék = `docs-sync` HEAD).
- Tiszta → commit + push a `docs-sync`-re. Ütközés → **barátságos felbontó** („az övék" vs. „a tiéd", blokkonként), **soha nyers `<<<<<<<`**.
- Commit **`--author` = szerző**, committer = bot.
- A Publish **csak** a `docs-sync`-re commitol — **nem** nyit PR-t (az E12, külön lépés).
- Az AI a piszkozatba ír → nincs külön konfliktusfelület.

## Taskok

- [ ] **E11-T1** Publish-orkesztráció: `fetch docs-sync` → 3-way merge a bázis-SHA-hoz képest (git `merge-file`, E06-T8).
- [ ] **E11-T2** **Tiszta eset** (bázis = docs-sync HEAD) → egyszerű commit + push. **MRG-01**.
- [ ] **E11-T3** **Bázis elmozdult, diszjunkt változás** → automata merge, tiszta. **MRG-02**.
- [ ] **E11-T4** **Átfedő változás** → konfliktus detektálás → barátságos felbontó adatstruktúra (blokkonkénti „övék/tiéd"). **MRG-03**.
- [ ] **E11-T5** **Új dokumentum** (nincs bázis-SHA, `target_path`) → új `.md` fájl commit a `docs-sync`-re + egyedi id (E07-T13). **MRG-05**.
- [ ] **E11-T6** **Konfliktus-feloldás → újrapróba** → sikeres commit. **MRG-04**.
- [ ] **E11-T7** **Publish közbeni verseny** (docs-sync újra mozdul) → újra-fetch + retry. **MRG-06**.
- [ ] **E11-T8** Commit **`--author` = szerző**, committer = bot; ellenőrzés a git-objektumon. **MRG-07**.
- [ ] **E11-T9** A **frontmatter-komment megőrzése** a merge során (kötődik FM-02/E08-T10). **MRG-08**.
- [ ] **E11-T10** **Nyers marker tilalom:** a konfliktus **sosem** ér nyers `<<<<<<<`-ként a felhasználóhoz (e2e a felbontón). **MRG-09**.
- [ ] **E11-T11** Draft-kép commitolása a Publish részeként (E09-T14): asset stabil útvonalra + hivatkozás-átírás + egy commitban a doksival. **IMG-02**.
- [ ] **E11-T12** Publish utáni takarítás: lock felszabadítás (**LOCK-07**) + draft törlés (**DRAFT-08**) + `audit_event` írása (tartalom-mentes).
- [ ] **E11-T13** Commit-üzenet kezelése (előre kitöltött, szerkeszthető a UI-ból, E15/E16).
- [ ] **E11-T14** **Property tesztek:** véletlen párhuzamos szerkesztések → **nincs adatvesztés**, valid eredmény (**MRG-10**); merge után mindig valid frontmatter + valid Markdown (**MRG-11**).

## Kész-definíció

- Tiszta és diszjunkt merge automatikusan publikál (MRG-01/02).
- Átfedő változás barátságos felbontót ad, feloldás után újrapróba sikeres (MRG-03/04).
- Verseny esetén re-fetch + retry, sosem veszik adat (MRG-06/10).
- A commit author a szerző, committer a bot (MRG-07); nyers marker soha (MRG-09).

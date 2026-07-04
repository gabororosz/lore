# E08 — Markdown szerializáció (minimal-diff + passthrough)

**Forrás:** koncepció (Markdown; Szerkesztő: WYSIWYG ↔ Markdown szerializáció), technikai terv §1 (TipTap), §5 (load-bearing kockázat).
**Cél:** forrás-megőrző (minimal-diff) szerializáció + passthrough az ismeretlen konstrukciókra — a „reviewzható, mint a kód" ígéret load-bearing magja.
**Függőség:** E01. (Az editor-integráció E15, de a szerializációs mag önállóan tesztelhető.)
**Tesztek:** SER-01…14, FM-01, FM-02, SER-09/10 (wiki-link, kép node).

## Döntések (a tervből rögzítve)

- **Szerializációs stratégia: minimal-diff (forrás-megőrző)** — csak a ténylegesen módosított csomópontok íródnak újra, az érintetlen részek **bájtazonosak** maradnak. A naiv full-reparse (kanonizálás) **kizárva** (diff-zaj → megtörné a review-ígéretet).
- **Ismeretlen Markdown:** **passthrough-megőrzés** (átlátszó blokk, bájtre visszaírva) + **forrás-mód fallback**. Adat sosem veszik el.
- **Framework: ProseMirror-alap (TipTap)** — séma-vezérelt, egyedi node-ok (`[[wiki-link]]`, kép, passthrough). A séma **maga a „jól definiált Markdown-részhalmaz"**.
- Támogatott részhalmaz: CommonMark + GFM (táblázat, checklista, kódblokk), kép, link, idézet, frontmatter.

## Taskok

- [ ] **E08-T1** TipTap/ProseMirror séma a támogatott Markdown-részhalmazra (címsor, félkövér/dőlt, listák, checklista, idézet, kódblokk, GFM-táblázat, link, kép).
- [ ] **E08-T2** Egyedi node-ok: `[[wiki-link]]` (id + label), image (repo asset-path), **passthrough** (ismeretlen konstrukció bájt-szintű megőrzésére).
- [ ] **E08-T3** **Parse (Markdown → fa)** source-mappinggel: forráspozíciók megőrzése, hogy a minimal-diff visszaíráshoz legyen forrás↔fa leképezés.
- [ ] **E08-T4** **Serialize (fa → Markdown) minimal-diff módon:** az érintetlen csomópontok forrása betű szerint marad; csak a módosított node-ok íródnak újra. **SER-02, SER-13**.
- [ ] **E08-T5** Idempotencia: érintetlen dokumentum parse→serialize **bájtazonos**. **SER-01, SER-12**.
- [ ] **E08-T6** Stílus-megőrzés: `*`/`_` kiemelés, `-`/`*` felsorolás, ATX/setext címsor, behúzás, sortörés változatlan. **SER-03, SER-04, SER-05**.
- [ ] **E08-T7** Passthrough: nyers HTML-blokk, lábjegyzet, egzotikus kiterjesztés **bájtazonos** megőrzése (fuzzra is). **SER-06, SER-14**.
- [ ] **E08-T8** Kódblokk tartalma érintetlen (nincs benne markdown-értelmezés). **SER-07**. GFM-táblázat megőrzése szerkesztés után. **SER-08**.
- [ ] **E08-T9** `[[wiki-link]]` node → `[[...]]` visszaszerializálás. **SER-09**. Kép-node → helyes repo asset-útvonal. **SER-10**.
- [ ] **E08-T10** **Frontmatter-kezelés komment-megőrzéssel:** a törzs szerkesztése nem írja át a frontmattert (**SER-11**); az `id` megőrződik (**FM-01**); a figyelmeztető YAML-komment **túléli** az újraszerializálást (**FM-02**).
- [ ] **E08-T11** **Property/fuzz tesztek:** véletlen valid Markdown → round-trip idempotens (**SER-12**); bármely szerkesztés után az érintetlen régiók bájtazonosak (**SER-13**); tetszőleges ismeretlen konstrukció megőrződik (**SER-14**). Golden-file (snapshot) tesztek.

## Kész-definíció

- Érintetlen dokumentum round-tripje bájtazonos (SER-01).
- Egy blokk módosítása után **csak** az a blokk változik a diffben (SER-02).
- Ismeretlen Markdown bájtre megőrződik (SER-06/14); a frontmatter-komment túléli (FM-02).
- A property/fuzz tesztek zölden futnak (coverage-kapu, E03-T9).

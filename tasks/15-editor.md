# E15 — WYSIWYG szerkesztő (TipTap) + új dokumentum + AI-panel

**Forrás:** UX-terv (felület 4 új dokumentum, felület 6 szerkesztő + AI, felület 7 Publish-panel), koncepció + technikai terv §5 (szerkesztő).
**Cél:** a mobile-first WYSIWYG szerkesztő a piszkozaton, forrás-mód fallbackkel, wiki-link/kép/metaadat kezeléssel, AI-panellel, Publish-belépővel.
**Függőség:** E08 (szerializációs mag), E09 (draft/autosave), E10 (lock/presence), E13 (AI), E11 (Publish).
**Tesztek:** MRG-05 (új doksi), IMG-01/02, MOB-01…04, SER-* (a szerializáció E08).

## Döntések (a tervből rögzítve)

- **Framework: TipTap (ProseMirror)** — a séma a támogatott Markdown-részhalmaz (E08).
- **Forrás-mód fallback** a sémán kívüli / passthrough tartalomhoz.
- **Mobile-first, IME/érintőbevitel első osztályú** — valódi eszközön validálandó (a frameworkök itt térnek el legjobban).
- Az `id`-t **sosem** írja a felhasználó (Réteg 4 megelőzés, E07-T13); látható, de nem szerkeszthető.
- Az AI a **draftba** ír, jelölt módosítással; a felhasználó hagyja jóvá.

## Taskok — Új dokumentum (felület 4)

- [ ] **E15-T1** Új dokumentum űrlap: `Cím` (kötelező), `Hely`/mappa-picker („új mappa" opcióval), útvonal-előnézet (`docs/…/slug.md`), `Tulajdonos` (default aktuális user), `Címkék`. Draftként jön létre → szerkesztő. Egyedi `id` háttérben (E07-T13), Publish-kor új fájl (MRG-05).

## Taskok — Szerkesztő (felület 6)

- [ ] **E15-T2** TipTap WYSIWYG a draften; formázás: címsorok, félkövér/dőlt, listák (rendezett/rendezetlen), checklista, idézet, kódblokk (nyelvvel), táblázat, link, kép.
- [ ] **E15-T3** **Wiki-link beszúró:** cím szerinti keresés (index, E07), a szövegbe stabil `id` kerül, a label a cím.
- [ ] **E15-T4** **Kép beszúrás:** feltöltés draft-assetként (E09-T13), azonnali előnézet; végső repo-útvonal Publish-kor. **IMG-02**.
- [ ] **E15-T5** **Forrás-mód váltó** (`WYSIWYG ↔ Forrás`): nyers Markdown a sémán kívüli / passthrough / fidelitás-kritikus tartalomhoz (E08-T2/T7).
- [ ] **E15-T6** **Metaadat-űrlap** (frontmatter): `title`, `owner`, `status`, `tags`; az `id` látható, **nem szerkeszthető**, figyelmeztető megjegyzéssel.
- [ ] **E15-T7** **Autosave-állapot** kijelzés („Mentve"/„Mentés…"/„Offline — helyileg mentve"), az E09 autosave-re kötve.
- [ ] **E15-T8** **Lock + presence** a szerkesztőben: saját lock aktív jelzése + lejárat-közeledés halk figyelmeztetése („Még szerkesztesz? … `Folytatom`"); presence-avatarok (E10). Konkurencia: foglalt doksi read-only, nem nyílik szerkesztőben.
- [ ] **E15-T9** „A publikált verzió változott" halk jelzés, ha a `docs-sync` a szerkesztés alatt elmozdult (bázis-SHA vs. HEAD).

## Taskok — AI-panel (felület 6)

- [ ] **E15-T10** AI-panel: utasítás-mező + `Küldés`; eredmény **streamelve** épül a draftba (E13/E10). Mobilon teljes képernyős/alulról csúszó, desktopon jobb oszlop.
- [ ] **E15-T11** AI-eredmény kezelése: `Elfogad`/`Elvet`/`Finomít`; a módosítás jelölt (mi változott).

## Taskok — Publish-belépő (felület 7 — a merge E11)

- [ ] **E15-T12** `Publish` panel/bottom sheet: összegzés (doksi címe, cél `docs-sync`), szerkeszthető commit-üzenet, `Közzététel`/`Mégse`. Siker-toast + read-nézet frissülés.
- [ ] **E15-T13** `Elvetés` (draft eldobás → azonnali törlés, E09-T10); kilépés mentéssel (draft marad, lock lejár policy szerint).

## Taskok — Mobil-validáció

- [ ] **E15-T14** **Manuális IME/érintő-kör** valódi eszközön: iOS Safari (**MOB-01**), Android Chrome (**MOB-02**), forrás-mód ↔ WYSIWYG váltás mobilon (**MOB-03**). Playwright mobil-viewport a fő hurokra (**MOB-04**).
- [ ] **E15-T15** Mobil eszköztár: a billentyűzet fölé dokkolt kontextsáv; desktopon rögzített toolbar.

## Kész-definíció

- Új doksi létrehozható, szerkeszthető, publikálható (MRG-05).
- WYSIWYG + forrás-mód + wiki-link + kép + metaadat működik; az `id` nem szerkeszthető.
- Az AI a draftba streamel, a módosítás jelölt és jóváhagyható.
- A manuális mobil IME-kör lefutott iOS+Android eszközön (MOB-01/02).

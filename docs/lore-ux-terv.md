# Lore — UX-terv

*Kiegészítő dokumentum a Lore projektkoncepcióhoz és a technikai tervhez. A koncepció a „mit és miért", a technikai terv a „mivel és hogyan"; ez a dokumentum a „mit lát és mit csinál a felhasználó": a felületek, a rajtuk megjelenő információk és funkciók, a konkrét UI-elemek, gombok és mezők. Minden itt rögzített döntés az MVP-re optimalizált, iterálható, és a koncepció elveire épül (mobile-first, read-first Draft→Publish, Git-native igazságforrás, cserélhető rétegek).*

---

## Vezérelvek (a UX magja)

Minden képernyő- és komponensdöntés az alábbi öt elvből vezethető le. Ha egy UI-ötlet ezekkel ütközik, az figyelmeztető jel.

1. **Read-first.** Minden dokumentum olvasásra nyílik meg. A szerkesztés szándékos, külön `Edit` gombbal kezdeményezett állapot — nincs véletlen szerkesztés.
2. **Mobile-first.** A layout, az érintési célpontok (min. 44×44 pt), a bevitel és a navigáció elsődlegesen mobilra tervezett; a desktop a mobil kiterjesztése, nem fordítva.
3. **Apple Notes / Obsidian egyszerűség.** A felhasználó a projektet nyitja meg, és azonnal a dokumentációt látja — nem repo-böngészőt, nem Git-fogalmakat. A Git a felszín alatt van.
4. **A Git-állapot barátságosan látszik, nyers Git sosem.** A publikálatlanság, a review-állapot, a konfliktus mind emberi nyelven jelenik meg (badge, felbontó) — soha nem nyers `<<<<<<<` marker, SHA vagy branch-parancs.
5. **Az AI a piszkozatba ír, az ember hagyja jóvá.** Az AI sosem publikál közvetlenül; a kimenete a draftba folyik, a felhasználó látja épülni, és ő nyom `Publish`-t.

---

## Információs architektúra és navigáció

### A navigáció három szintje

```
Belépés (GitHub OAuth)
  ↓
Projekt-választó  (mely bekötött repók érhetők el)
  ↓
Projekt-nézet  (a /docs fa + kereső + függő piszkozatok)
  ↓
Dokumentum-nézet  (read → edit → publish)
```

### Elsődleges navigációs minta

- **Mobil:** alsó navigációs sáv (thumb-zónában) + a dokumentum-fa lenyíló/oldalról becsúszó panelként. A tartalom kapja a teljes szélességet.
- **Desktop:** háromoszlopos elrendezés — bal: dokumentum-fa; közép: tartalom (read/edit); jobb: kontextuspanel (AI, tartalomjegyzék, dokumentum-infó), amely elrejthető.

### Globális, mindig elérhető elemek

- **Kereső** (a projekt-nézet tetején, mobilon ikonból nyíló teljes képernyős kereső).
- **Felhasználói menü** (avatar): fiók, aktív projekt váltása, függő piszkozatok, kijelentkezés.
- **Új dokumentum** gomb (mobilon úszó FAB `+`, desktopon a fa fejlécében).
- **Kapcsolat/szinkron-állapot** diszkrét jelző (online / offline-piszkozat cache / szinkron folyamatban).

---

## Felületek (képernyők)

Az alábbi szakaszok képernyőnként sorolják fel a **megjelenő információt**, a **funkciókat**, és a **konkrét UI-elemeket** (gombok, mezők, jelzők).

---

### 1. Belépés / onboarding

**Cél:** GitHub-alapú azonosítás és az első projekt bekötése.

**Megjelenő információ:**
- Rövid termékállítás (egy mondat: „A projekted tudása a repóban, mobilról is szerkeszthetően").
- Adatkezelési/adatvédelmi rövid utalás (link a DPA/adatkezelési tájékoztatóhoz).

**Funkciók:**
- Bejelentkezés GitHubbal (OAuth).
- Első belépéskor: a Lore GitHub App telepítésének indítása a kiválasztott repókra (a tenant/admin számára).

**UI-elemek:**
- `Bejelentkezés GitHubbal` gomb (elsődleges).
- `A Lore GitHub App telepítése` gomb / átirányítás (onboarding lépés).
- Lábléc-linkek: Adatvédelem, Feltételek.

**Állapotok:** betöltés (OAuth-átirányítás), hiba (megtagadott jogosultság → magyarázó üzenet, újrapróba).

---

### 2. Projekt-választó

**Cél:** a felhasználó számára elérhető, bekötött repók (projektek) listája és váltás közöttük.

**Megjelenő információ (projektkártyánként):**
- Projekt (repo) neve, tulajdonos/org.
- Láthatóság (privát / internal / publikus) jelvény.
- Utolsó aktivitás (utolsó publish ideje).
- Jelzés, ha a felhasználónak van **függő piszkozata** ebben a projektben (számláló).
- Állapotjelzés, ha a repo még szinkronizál (első mirror-építés folyamatban).

**Funkciók:**
- Projekt megnyitása.
- Új repo bekötése (ha jogosult): a GitHub App telepítése/kiterjesztése.
- Keresés a projektek között (ha sok van).

**UI-elemek:**
- Projekt-kártyalista (mobilon egy oszlop).
- `Új projekt bekötése` gomb.
- Kártyán: láthatóság-badge, „N függő piszkozat" pill.

**Üres állapot:** ha nincs bekötött repo → magyarázat + `Repo bekötése` elsődleges gomb.

---

### 3. Projekt-nézet (dokumentum-fa + kereső)

**Cél:** a `/docs` struktúra böngészése, keresés, és a belépési pont a dokumentumokba. Ez a „home" a projekten belül.

**Megjelenő információ:**
- A `/docs` **könyvtárfa** az eredeti struktúrával (mappák: product, architecture, backend, …), a fájlok **cím** (frontmatter `title`) szerint, nem nyers fájlnévvel.
- Dokumentumonkénti **állapot-badge** (lásd külön szakasz): szinkronban / friss verzió / review alatt.
- **Probléma-jelzés** ikon a hibás dokumentumokon (⚠ érvénytelen frontmatter, duplikált id, törött link).
- Zárolás-jelző, ha egy dokumentumot épp más szerkeszt („X szerkeszti").
- **Függő piszkozatok** szekció/belépő (a saját befejezetlen munkák).

**Funkciók:**
- Fa böngészése, mappák nyitása/csukása.
- Dokumentum megnyitása (→ read-nézet).
- Keresés (cím + tartalom, Postgres FTS).
- Új dokumentum létrehozása (→ 4. felület).
- Rendezés/szűrés (pl. státusz szerint, opcionális).

**UI-elemek:**
- Kereső mező (fejlécben; mobilon ikonból teljes képernyős).
- Fa-nézet nyíló csomópontokkal; leveleken cím + badge + esetleg ⚠.
- `+ Új dokumentum` (FAB mobilon / fejléc-gomb desktopon).
- „Függő piszkozataim" belépő (számlálóval).
- Breadcrumb a mélyebb mappákban.

**Üres állapot:** ha a `/docs` üres vagy nincs → magyarázat + `Első dokumentum létrehozása` gomb.

---

### 4. Új dokumentum létrehozása

**Cél:** új Markdown-dokumentum felvétele a `/docs` alá, a Draft→Publish modellbe illesztve (a koncepció MVP-funkciója; a technikai tervben a draft `target_path`-szal, tesztben MRG-05).

**Megjelenő információ:**
- Cél könyvtár (hol jön létre) — előre kitöltve az aktuálisan böngészett mappával.
- Az automatikusan generált fájl-útvonal **előnézete** (a címből képzett slug alapján).
- Jelzés, hogy a dokumentum **draftként** jön létre, és csak `Publish`-kor kerül a `docs-sync`-re.

**Funkciók:**
- Cím megadása (ebből generálódik a slug/path és a frontmatter `title`).
- Cél mappa kiválasztása/váltása (a meglévő fából, vagy új mappa megadása).
- Automatikus, **egyedi** frontmatter `id` generálása (slug + rövid random suffix) — a háttérben, nem szerkeszthető mezőként, de látható.
- Opcionális kezdő-metaadat: `owner` (default a belépett user), `tags`.
- Létrehozás → azonnal a szerkesztő-nézetbe (5.), új draft-tal.

**UI-elemek:**
- `Cím` szövegmező (kötelező).
- `Hely` / mappa-választó (fa-picker vagy útvonal-mező, „új mappa" opcióval).
- Útvonal-előnézet (csak olvasható, pl. `docs/backend/uj-dokumentum.md`).
- `Tulajdonos` (owner) választó (default: aktuális user).
- `Címkék` (tag) beviteli mező (opcionális).
- `Létrehozás és szerkesztés` elsődleges gomb; `Mégse` másodlagos.

**Következmény, amit ez rögzít:** az `id` generálása a Lore feladata (invariáns-megelőzés, „Réteg 4"), a felhasználó sosem ír `id`-t kézzel. Az útvonal átnevezhető később anélkül, hogy a linkek törnének (az identitás az `id`).

---

### 5. Dokumentum — olvasónézet (read)

**Cél:** a dokumentum megjelenítése, a Git-állapot barátságos kommunikálása, és a belépés a szerkesztésbe.

**Megjelenő információ:**
- A renderelt Markdown (címsorok, listák, táblázatok, kódblokkok, képek, idézetek, checklisták).
- Feloldott **wiki-linkek** (a cím-label jelenik meg, kattintható; törött link „törött" jelöléssel).
- Dokumentum-fejléc: cím, `owner`, `status`, `tags`, utolsó módosítás (szerző + idő a Git author alapján).
- **Állapot-badge** (`main`-hez viszonyítva): szinkronban / friss verzió / review alatt.
- Ha más szerkeszti: „X szerkeszti" jelzés + read-only kontextus.
- Ha a dokumentumnak **problémája** van (⚠): látható, nem elrejtett jelölés.

**Funkciók:**
- Olvasás, görgetés, wiki-linkek követése.
- `Edit` — lock kérése és belépés a szerkesztőbe (ha szabad).
- `Propose to main` / PR-állapot megnyitása (ha van nyitott PR: link a review-hoz).
- Megosztás/másolás (link a dokumentumra a Lore-on belül).
- Tartalomjegyzék (hosszú dokumentumnál, a kontextuspanelen vagy lenyílóban).
- Verziótörténet megnyitása (Git-történet barátságos nézete — legalább az utolsó módosítások szerzővel; részletes diff a `main` vs `docs-sync` badge-hez kötve).

**UI-elemek:**
- `Edit` elsődleges gomb (kiemelt, thumb-zónában mobilon).
- Állapot-badge a fejlécben (kattintható → magyarázat / PR-link).
- Kontextusmenü (`⋯`): Propose to main, Verziótörténet, Link másolása, Dokumentum-infó.
- „X szerkeszti" bannernél: `Csak olvasom` (implicit) — az `Edit` letiltva vagy „Átvétel kérése" (ha a lock lejárt/elhagyott).

**Állapotok:** betöltés, hiba (nincs jogosultság → magyarázat), degradált (érvénytelen frontmatter → nyers, olvasható tartalom „⚠ probléma" jelöléssel).

---

### 6. Dokumentum — szerkesztő (edit) + AI

**Cél:** a WYSIWYG szerkesztés a piszkozaton, forrás-mód fallbackkel, AI-segítséggel, lock alatt.

**Megjelenő információ:**
- A szerkeszthető tartalom (WYSIWYG, TipTap/ProseMirror alap).
- **Autosave-állapot** („Mentve", „Mentés…", „Offline — helyileg mentve").
- A saját **lock** aktív jelzése + a lock-lejárat közeledtének halk figyelmeztetése.
- Presence: ki nézi épp a dokumentumot (avatarok).
- Bázis-állapot jelzése, ha közben a `docs-sync` elmozdult (halk „a publikált verzió változott" jelzés a Publish előtt).

**Funkciók — szerkesztés:**
- Formázás: címsorok, félkövér/dőlt, listák (rendezett/rendezetlen), checklista, idézet, kódblokk (nyelvvel), táblázat, link, kép.
- **Wiki-link beszúrása:** cím szerinti kereséssel (az index adja a találatokat), a háttérben stabil `id` kerül a szövegbe, a label a cím.
- **Kép beszúrása:** feltöltés (draft-asset), azonnali előnézet, a végső repo-útvonal Publish-kor.
- **Forrás-mód** váltás (nyers Markdown) — a sémán kívüli / fidelitás-kritikus tartalomhoz és a passthrough-blokkokhoz.
- **Metaadat-szerkesztés** (frontmatter): `title`, `owner`, `status`, `tags` űrlapon (az `id` látható, de nem szerkeszthető, figyelmeztető megjegyzéssel).

**Funkciók — AI (a piszkozaton):**
- Utasítás-alapú szerkesztés a `/docs` tartalmán (átírás, összefoglalás, kiegészítés, javítás).
- Az eredmény **streamelve** épül a draftba (SSE), a felhasználó látja készülni.
- Az AI-módosítás **jelölt** (mi változott), a felhasználó elfogadhatja/elvetheti/finomíthatja.

**Funkciók — kilépés/publikálás:**
- `Publish` → a Publish-folyamat (7.).
- `Elvetés` (draft eldobása → azonnali törlés).
- Kilépés mentéssel (a draft megmarad, a lock a policy szerint lejár).

**UI-elemek:**
- Formázó eszköztár (mobilon a billentyűzet fölötti kontextsáv; desktopon rögzített toolbar).
- `WYSIWYG ↔ Forrás` váltókapcsoló.
- `AI` gomb/panel: utasítás-mező + `Küldés`; eredményen `Elfogad` / `Elvet` / `Finomít`.
- Autosave-státusz felirat (nem gomb).
- `Publish` elsődleges gomb; `Elvet` és `Metaadat` másodlagos.
- Lock-lejárat közeledtekor: „Még szerkesztesz? A zárolás X perc múlva lejár" + `Folytatom`.

**Konkurencia:** ha valaki más tartja a lockot, ez a felület read-only formában nem is nyílik meg (a read-nézet mutatja a „X szerkeszti" állapotot).

---

### 7. Publish + konfliktus-felbontó

**Cél:** a piszkozat közzététele a `docs-sync`-re, bázis-SHA + 3-way merge-dzsel; konfliktus esetén barátságos feloldás.

**Megjelenő információ (Publish előtt):**
- Rövid összegzés: mit publikálsz (dokumentum címe, cél branch: `docs-sync`).
- Commit-üzenet mező (előre kitöltött, szerkeszthető).
- Jelzés, ha a `docs-sync` időközben elmozdult (merge szükséges lehet).

**Funkciók:**
- Publikálás (commit `--author` = felhasználó, push a `docs-sync`-re a bot által).
- Konfliktus esetén felbontás.
- Publikálás utáni továbblépés: „Propose to main" felkínálása (nem kötelező).

**UI-elemek — sikeres (tiszta) ág:**
- `Publish` megerősítő panel: commit-üzenet mező, `Közzététel` gomb, `Mégse`.
- Siker-visszajelzés (toast + a read-nézet frissült badge-dzsel „friss verzió").
- Felkínálás: `Javaslom a main-be` gomb (opcionális, külön lépés).

**UI-elemek — konfliktus ág (barátságos felbontó):**
- Kétoldalas összevetés: **„Az övék" (a publikált, időközben változott verzió)** vs. **„A tiéd" (a piszkozatod)** — blokkonként.
- Blokkonkénti választó: `Az övék`, `A tiéd`, vagy `Mindkettő/szerkesztem`.
- **Soha nem** nyers `<<<<<<<` markerek.
- `Feloldás és újrapróba` gomb; `Mégse` (vissza a szerkesztőbe, a draft megmarad).

**Következmény:** ez az egyetlen pont, ahol Git-konfliktus felszínre kerülhet — a nem-technikai célközönség itt sem lát nyers Git-fogalmat.

---

### 8. Propose to main (PR) + review-állapot

**Cél:** a `docs-sync` → `main` Pull Request nyitása és állapotkövetése — a „dokumentáció reviewzható, mint a kód" belépője.

**Megjelenő információ:**
- Mely dokumentum-változások kerülnek a PR-be (a `docs-sync` a `main` felett — akár több publish együtt).
- PR-cím és leírás (szerkeszthető).
- Nyitott PR állapota (nyitva / review alatt / mergelve / elutasítva), link a GitHub PR-re.

**Funkciók:**
- PR nyitása a `main` felé.
- Meglévő nyitott PR megtekintése (link + állapot-badge).

**UI-elemek:**
- `Javaslom a main-be` gomb (a read-nézetből és a Publish után).
- PR-űrlap: `Cím`, `Leírás` mezők, `PR létrehozása` gomb.
- PR-állapot pill a dokumentum fejlécében („review alatt / javasolt"), kattintva → GitHub.

**Következmény:** a Publish és a Propose-to-main tudatosan külön van — a UI sem keveri; a Publish nem nyit automatikusan PR-t.

---

### 9. Keresés

**Cél:** dokumentum gyors megtalálása cím és tartalom alapján.

**Megjelenő információ:**
- Találatok: cím, útvonal/mappa, rövid kivonat a találati kontextussal, állapot-badge.
- Üres/nincs találat állapot.

**Funkciók:**
- Szöveges keresés (Postgres FTS az MVP-ben).
- Találatra ugrás (read-nézet).
- (Opcionális, iterálható: szűrés státusz/tag/owner szerint.)

**UI-elemek:**
- Kereső mező (fejléc / mobilon teljes képernyős overlay).
- Találati lista, kiemelt egyezésekkel.

---

### 10. Függő piszkozataim

**Cél:** a felhasználó lássa és kezelje a saját befejezetlen munkáit (a retenció miatt is fontos: ne érje meglepetésként a törlés).

**Megjelenő információ:**
- Piszkozatonként: dokumentum címe (vagy új doksinál a tervezett cím/útvonal), utolsó módosítás, **retenciós állapot** (aktív / szunnyadó — „X nap múlva törlődik").
- Jelzés, ha új dokumentum (még sosem publikált) vagy meglévő szerkesztése.

**Funkciók:**
- Piszkozat folytatása (→ szerkesztő).
- Piszkozat elvetése (azonnali törlés).
- Több eszköz közti folytatás (ugyanaz a draft másik eszközön).

**UI-elemek:**
- Piszkozat-lista, retenciós állapot-pill (szunnyadónál figyelmeztető szín + „N nap").
- Soronként: `Folytatom`, `Elvetem`.

---

### 11. Beállítások / admin

**Cél:** tenant- és projekt-szintű konfiguráció (elsősorban admin), felhasználói preferenciák.

**Megjelenő információ / funkciók:**
- **Retenciós szabály** (tenant-szintű, default ~30 nap; lefelé állítható) — enterprise/compliance igény.
- **Repo-bekötések** kezelése (mely repók, GitHub App installation állapota).
- `docs_path` megerősítése/felülírása (ha nem a default `/docs`).
- Felhasználói preferenciák: megjelenés (világos/sötét), nyelv (ha releváns), presence láthatósága.
- Adatkezelés: adat-export/-törlés belépő (GDPR), DPA-tájékoztató.

**UI-elemek:**
- Retenció-beállító (napok mezője + magyarázat).
- Repo-lista kezelőgombokkal.
- Preferencia-kapcsolók (téma, presence).

**Jogosultság:** a tenant-szintű beállítások csak adminnak; a sima felhasználó a saját preferenciáit és a saját piszkozatait látja.

---

## Állapot-badge rendszer (a Git-állapot barátságos nyelve)

A `docs-sync`-ből olvasunk, ezért minden dokumentum állapotát a `main`-hez viszonyítva jelöljük. Ez a UI-ban egységes, visszafogott badge-rendszer:

| Állapot | Jelentés | Vizuális súly |
|--------|----------|---------------|
| **Szinkronban** | nincs függő változás, minden a `main`-en van | **nincs** badge (nyugalmi) |
| **Friss verzió** | publikálva a `docs-sync`-re, nincs nyitott PR | halk, informatív badge |
| **Review alatt / javasolt** | nyitott PR a `main` felé | badge + opcionális PR-link |
| **⚠ Probléma** | érvénytelen frontmatter / duplikált id / törött link | figyelmeztető ikon, nem elrejtve |
| **Zárolt** | más user épp szerkeszti | „X szerkeszti" jelzés |

**Elv:** a „friss verzió" a *gyakori* köztes állapot, ezért a badge szándékosan visszafogott — nincs „mindig sárga a képernyő" hatás. A ⚠ viszont mindig látszik (graceful degradation: a hiba nem rejtőzik el, de nem is dönti össze a nézetet).

---

## Degradált / hibaállapotok a UI-ban

A koncepció „detektálás + graceful degradation" elve konkrét UI-viselkedéssé fordul:

- **Érvénytelen frontmatter:** a dokumentum nyersen, olvasható formában jelenik meg, „⚠ probléma" jelöléssel; nem tűnik el, nem crashel.
- **Duplikált id:** mindkét dokumentum látszik, ütközés-jelöléssel; a wiki-linkek a régebbi commit doksijára oldódnak (a UI ezt jelzi az újabbon).
- **Törött wiki-link:** „törött" (404-szerű) render a szövegben; a környező tartalom **nem** tűnik el.
- **Lock elhagyott/lejárt:** a read-nézet felkínálja az `Átvétel`-t; a korábbi user piszkozatához nincs hozzáférés.
- **Offline (mobil):** a piszkozat helyben (PWA-cache) mentődik, a státusz „Offline — helyileg mentve"; a szinkron a kapcsolat visszatértekor.

*Fontos:* a Lore nem javít automatikusan (minden írás attribútált commit) — a vezetett javítás post-MVP, az MVP-ben a UI csak láthatóvá tesz és nem omlik össze.

---

## Globális UI-komponensek leltára

Újrafelhasználható komponensek, amelyek több felületen megjelennek:

- **Navigációs sáv** (mobil: alsó; desktop: bal oldali/fejléc).
- **Dokumentum-fa** (nyíló csomópontok, cím + badge + ⚠).
- **Állapot-badge** (a fenti rendszer szerint).
- **Presence-avatarok** (ki nézi / ki szerkeszti).
- **FAB `+`** (új dokumentum — mobil).
- **Kontextusmenü `⋯`** (dokumentum-műveletek).
- **Toast/snackbar** (siker, hiba, autosave-visszajelzés).
- **Megerősítő panel / bottom sheet** (Publish, Elvetés, PR — mobilon alulról csúszó lap).
- **Kereső overlay**.
- **Konfliktus-felbontó** (kétoldalas, blokkonkénti választóval).
- **Metaadat-űrlap** (frontmatter-mezők).
- **AI-panel** (utasítás-mező + streamelő kimenet + elfogad/elvet).

---

## Gomb- és mező-katalógus (elsődleges akciók)

| Akció | Elem | Hol | Típus |
|-------|------|-----|-------|
| Bejelentkezés | `Bejelentkezés GitHubbal` | Belépés | elsődleges gomb |
| Projekt megnyitása | kártya-tap | Projekt-választó | navigáció |
| Új dokumentum | `+ Új dokumentum` / FAB | Projekt-nézet | elsődleges gomb |
| Cím megadása | `Cím` | Új dokumentum | szövegmező (kötelező) |
| Hely kiválasztása | `Hely` / mappa-picker | Új dokumentum | választó |
| Szerkesztés indítása | `Edit` | Read-nézet | elsődleges gomb |
| Formázás | eszköztár-ikonok | Szerkesztő | ikon-gombok |
| Wiki-link | `[[ ]]` beszúró | Szerkesztő | kereső-beszúró |
| Kép | kép-ikon | Szerkesztő | feltöltő |
| Forrás-mód | `WYSIWYG ↔ Forrás` | Szerkesztő | kapcsoló |
| AI-utasítás | utasítás-mező + `Küldés` | AI-panel | mező + gomb |
| AI-eredmény kezelése | `Elfogad` / `Elvet` / `Finomít` | AI-panel | gombok |
| Metaadat | `Metaadat` | Szerkesztő | űrlap-megnyitó |
| Közzététel | `Publish` → `Közzététel` | Szerkesztő / Publish-panel | elsődleges gomb |
| Piszkozat eldobása | `Elvet` | Szerkesztő / Piszkozataim | destruktív gomb |
| Konfliktus feloldása | `Az övék` / `A tiéd` / `Mindkettő` + `Feloldás és újrapróba` | Konfliktus-felbontó | választók + gomb |
| PR nyitása | `Javaslom a main-be` → `PR létrehozása` | Read-nézet / Publish után | gomb + űrlap |
| Keresés | kereső mező | Projekt-nézet | szövegmező |
| Piszkozat folytatása | `Folytatom` | Piszkozataim | gomb |
| Retenció beállítása | napok mező | Beállítások (admin) | szám-mező |

---

## Mobil vs. desktop — a fő eltérések

- **Navigáció:** mobilon alsó sáv + becsúszó fa; desktopon állandó bal fa + jobb kontextuspanel.
- **Eszköztár:** mobilon a billentyűzet fölé dokkolt kontextsáv; desktopon rögzített toolbar.
- **Panelek:** mobilon bottom sheet (Publish, PR, metaadat); desktopon oldalpanel/modal.
- **Új dokumentum:** mobilon FAB; desktopon fejléc-gomb.
- **AI-panel:** mobilon teljes képernyős/alulról csúszó; desktopon jobb oszlop.
- **Bevitel (kritikus):** az IME/érintőbeviteli viselkedés (autocorrect, kompozíció) valódi eszközön validálandó — ez a szerkesztő legkockázatosabb UX-pontja (lásd teszt: MOB-01…03).

---

## Nyitott, iterálható UX-részletek

Ezek tudatosan nyitva maradnak, a keret adott, a finomhangolás későbbi:

- A badge-ek pontos szövege és vizuális súlya; a „review alatt" mutasson-e mindig PR-linket.
- A verziótörténet-nézet mélysége (mennyi Git-diff jelenjen meg barátságosan a nem-technikai usernek).
- A forrás-mód ↔ WYSIWYG váltás UX-e mobilon.
- A wiki-link-beszúró és a kereső egyesítése (egy „gyors-kereső", ⌘K-szerű, mindenre).
- A presence granularitása (avatarok száma, „olvassa" vs. „szerkeszti" megkülönböztetés vizualizációja).
- Az onboarding lépésszáma (repo-bekötés súrlódásának csökkentése).
- A retenciós figyelmeztetés csatornája (in-app vs. e-mail a szunnyadó piszkozatról).

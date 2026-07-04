# E14 — Frontend app shell + PWA + navigáció

**Forrás:** UX-terv (Vezérelvek, IA és navigáció, felületek 1–3, 9–11, Globális komponensek).
**Cél:** a mobile-first PWA-váz: belépés, projekt-választó, projekt-nézet (dokumentum-fa + kereső), keresés, függő piszkozatok, beállítások — a globális navigációval és komponensekkel.
**Függőség:** E04 (auth), E07 (index/keresés/fa adatai), E09 (piszkozat-lista), E01-T5 (PWA-váz).
**Tesztek:** AUTH-01, READ-* (badge megjelenítés E16), keresés (FTS), IMG-01 (read-only megnyitás).

## Vezérelvek (a tervből rögzítve)

Read-first · Mobile-first (érintési célok min. 44×44 pt) · Apple Notes/Obsidian egyszerűség (nincs repo-böngésző, nincs Git-fogalom a felszínen) · a Git-állapot barátságosan látszik, nyers Git soha · az AI a piszkozatba ír.

## Taskok — Váz és navigáció

- [ ] **E14-T1** Navigáció 3 szintje: Belépés → Projekt-választó → Projekt-nézet → Dokumentum-nézet. Route-ok + guardok (auth).
- [ ] **E14-T2** Mobil navigációs minta: alsó sáv (thumb-zóna) + becsúszó dokumentum-fa panel; desktopon háromoszlopos (fa / tartalom / kontextuspanel).
- [ ] **E14-T3** Globális elemek: kereső, felhasználói menü (avatar), `+ Új dokumentum` (FAB mobil / fejléc desktop), kapcsolat/szinkron-állapot jelző.
- [ ] **E14-T4** Globális komponens-készlet: navigációs sáv, dokumentum-fa, presence-avatarok, FAB, kontextusmenü `⋯`, toast/snackbar, bottom sheet (mobil) / modal (desktop), kereső overlay. (Design system: `../design-system/`.)

## Taskok — Felületek

- [ ] **E14-T5** **1. Belépés/onboarding:** `Bejelentkezés GitHubbal` (OAuth, E04), GitHub App telepítés indítása, adatvédelmi láblécek. Betöltés/hiba állapotok. **AUTH-01**.
- [ ] **E14-T6** **2. Projekt-választó:** projektkártyák (név, owner, láthatóság-badge, utolsó publish, „N függő piszkozat" pill, mirror-szinkron állapot); új repo bekötése; üres állapot.
- [ ] **E14-T7** **3. Projekt-nézet:** `/docs` könyvtárfa (címekkel, nem fájlnévvel), dokumentumonkénti állapot-badge (E16), ⚠ probléma-jelzés, „X szerkeszti" zárolás-jelző, „Függő piszkozataim" belépő, breadcrumb, üres állapot.
- [ ] **E14-T8** **9. Keresés:** kereső mező/overlay (mobilon teljes képernyős); találatok (cím, útvonal, kivonat, badge); Postgres FTS-re kötve (E07-T4). Üres/nincs találat állapot.
- [ ] **E14-T9** **10. Függő piszkozataim:** piszkozat-lista retenciós állapot-pillel (aktív/szunnyadó „N nap"); `Folytatom`/`Elvetem`; multi-device folytatás (E09).
- [ ] **E14-T10** **11. Beállítások/admin:** retenció-beállító (napok, admin), repo-bekötések kezelése, `docs_path` felülírás, felhasználói preferenciák (téma, presence-láthatóság), GDPR export/törlés belépő. Jogosultság: tenant-beállítás csak adminnak.

## Taskok — PWA / mobil

- [ ] **E14-T11** PWA: telepíthetőség, offline-tűrő piszkozat-cache váz (a szinkron E09/E10-zel); „Offline — helyileg mentve" állapot.
- [ ] **E14-T12** Kapcsolat/szinkron-állapot jelző (online / offline-cache / szinkron folyamatban).

## Kész-definíció

- A felhasználó bejelentkezik, projektet választ, böngészi a `/docs` fát címek szerint, keres, és látja a függő piszkozatait.
- A navigáció mobilon thumb-barát, desktopon háromoszlopos; a globális komponensek újrafelhasználhatók.
- Nyers Git-fogalom sehol nem jelenik meg.

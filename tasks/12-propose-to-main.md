# E12 — Propose to main (PR) + review-állapot

**Forrás:** koncepció (Publish vs. Propose to main; Git integráció: read-forrás, háromállapotú badge), UX-terv (felület 8).
**Cél:** `docs-sync → main` Pull Request nyitása és állapotkövetés — a „dokumentáció reviewzható, mint a kód" belépője.
**Függőség:** E05 (GitHub App: Pull requests jog), E06 (git), E04 (`pull_request` séma).
**Tesztek:** GH-07 (PR nyitás), READ-05 (PR badge).

## Döntések (a tervből rögzítve)

- A **Publish és a Propose-to-main tudatosan külön** — a Publish nem nyit automatikusan PR-t → nincs PR-zaj, több publish gyűlhet egy PR-be.
- Következmény: a `docs-sync` jellemzően **tartósan előrébb jár** a `main`-nél — ez normális működés.
- A `pull_request.state` táplálja a read-nézet „review alatt / javasolt" badge-ét (E16).
- **`docs-sync` lifecycle-motor (visszaszinkron, elutasított PR, branch-protection stb.) = post-MVP** — itt csak PR-nyitás + állapot-tükrözés.

## Taskok

- [ ] **E12-T1** PR nyitása `docs-sync → main` a GitHub App-pal (Pull requests jog). **GH-07**.
- [ ] **E12-T2** PR-cím + leírás kezelése (előre kitöltött, szerkeszthető a UI-ból).
- [ ] **E12-T3** Meglévő nyitott PR detektálása/megtekintése: link + állapot; ne nyisson duplikált PR-t, ha már van nyitott.
- [ ] **E12-T4** `pull_request` rekord szinkronban tartása (nyitva / review alatt / mergelve / elutasítva) — webhook vagy lekérdezés alapján; a badge-hez (E16, READ-05).
- [ ] **E12-T5** A PR-be kerülő változások összegzése a UI-hoz (a `docs-sync` a `main` felett — akár több publish együtt).

## Kész-definíció

- Egy gombbal PR nyílik `docs-sync → main` (GH-07); nincs duplikált PR.
- A PR állapota tükröződik a `pull_request` táblában és a dokumentum-badge-en (READ-05).
- A Publish továbbra sem nyit automatikusan PR-t.

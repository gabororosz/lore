# Lore — Teszteset-katalógus

*A Technikai terv → Tesztelési stratégia végrehajtható kibontása. Kockázat-alapú: a **K (kritikus)** esetek ott vannak, ahol egy hiba adatvesztést vagy -korrupciót okoz; **N (normál)** a többi. Szintek: unit · property (property-based/fuzz) · integráció · kontraktus · e2e · manuális.*

**Azonosító-előtagok:** SER (szerializáció), FM (frontmatter), LINK (wiki-link), MRG (merge/publish), LOCK (lock/presence), DRAFT (piszkozat/retenció), GH (GitHub App), SSE (realtime), READ (read-forrás/badge), IMG (kép/olvasás), AUTH (auth), DEG (graceful degradation).

---

## 1. Markdown szerializáció — round-trip (SER)

A „reviewzható, mint a kód" ígéret load-bearing pontja: a minimál-diff nem formázhat át érintetlen tartalmat, a passthrough nem veszíthet el semmit.

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| SER-01 | Érintetlen dokumentum: parse → serialize **bájtazonos** (idempotencia) | unit/golden | K |
| SER-02 | Egyetlen bekezdés módosítása után **csak az az egy blokk** változik a diffben; minden más bájtazonos | unit/golden | K |
| SER-03 | Kiemelés-stílus megőrzése: `*` marad `*`, `_` marad `_` (nincs normalizálás) | unit | K |
| SER-04 | Felsorolás-jelölő megőrzése: `-` nem vált `*`-ra, a behúzás változatlan | unit | K |
| SER-05 | Címsor-stílus megőrzése: ATX marad ATX, setext marad setext | unit | N |
| SER-06 | Ismeretlen konstrukció (nyers HTML-blokk, lábjegyzet) **passthrough**: bájtazonos megőrzés | unit | K |
| SER-07 | Kódblokk tartalma érintetlen (nincs benne markdown-értelmezés) | unit | K |
| SER-08 | GFM-táblázat megőrzése szerkesztés után | unit | N |
| SER-09 | `[[wiki-link]]` node visszaszerializálása `[[...]]` formába | unit | K |
| SER-10 | Kép-node → helyes repo asset-útvonal | unit | N |
| SER-11 | Frontmatter-blokk változatlan, ha csak a törzs változik | unit | K |
| SER-12 | **Property:** véletlen valid Markdown-részhalmaz → round-trip idempotens | property | K |
| SER-13 | **Property:** bármely szerkesztés után az érintetlen régiók bájtazonosak | property | K |
| SER-14 | **Property:** tetszőleges ismeretlen konstrukció bájtre megőrződik (fuzz) | property | K |

---

## 2. Frontmatter-invariánsok (FM)

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| FM-01 | `id` megőrzése törzs-szerkesztéskor | unit | K |
| FM-02 | A figyelmeztető YAML-komment **túléli** az újraszerializálást (komment-megőrző kezelés) | unit | K |
| FM-03 | Valid YAML frontmatter helyes parse-olása (`id`, `title`, `owner`, `status`, `tags`) | unit | N |
| FM-04 | Érvénytelen YAML → **nincs crash**, `doc_issue: invalid_frontmatter` keletkezik | unit | K |
| FM-05 | Hiányzó `id` → `doc_issue: missing_id` | unit | K |
| FM-06 | Teljesen hiányzó frontmatter → kezelt, jelzett állapot | unit | N |
| FM-07 | Sérült `---` határolók → graceful kezelés | unit | N |
| FM-08 | Lore saját írásánál mindig **egyedi id generálása** (slug + random suffix) | unit | K |
| FM-09 | Publish előtti frontmatter-validáció a Lore saját írásánál | unit | K |

---

## 3. Wiki-link feloldás (LINK)

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| LINK-01 | `[[Payment API]]` feloldása **id alapján**, nem fájlnév alapján | unit | K |
| LINK-02 | A link túléli a céldokumentum **átnevezését** (id változatlan) | integráció | K |
| LINK-03 | A link túléli a cél **áthelyezését** másik könyvtárba | integráció | K |
| LINK-04 | Törött link (nem létező id) → `doc_issue: broken_link` + „törött" render | unit | N |
| LINK-05 | Duplikált cél-id: a link a **régebbi commit** doksijára oldódik | integráció | K |
| LINK-06 | `doc_link` gráfél helyes létrehozása/frissítése | unit | N |

---

## 4. Publish 3-way merge (MRG) — a legmagasabb kockázat

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| MRG-01 | Tiszta eset: bázis-SHA = docs-sync HEAD (nem mozdult) → egyszerű commit + push | integráció | K |
| MRG-02 | Bázis elmozdult, **diszjunkt** változás (más blokk) → automata merge, tiszta | integráció | K |
| MRG-03 | Bázis elmozdult, **átfedő** változás (ugyanaz a bekezdés) → konfliktus → barátságos felbontó | integráció | K |
| MRG-04 | Konfliktus feloldása után **újrapróba** → sikeres commit | integráció | K |
| MRG-05 | Új dokumentum (nincs bázis-SHA, `target_path`) → új fájl commit | integráció | K |
| MRG-06 | Publish közben a docs-sync **újra mozdul** (verseny) → újra-fetch + retry | integráció | K |
| MRG-07 | A commit **author = szerző**, committer = bot (ellenőrzés a git-objektumon) | integráció | K |
| MRG-08 | A frontmatter-komment megőrződik a merge során | integráció | K |
| MRG-09 | Konfliktus **sosem** eredményez nyers `<<<<<<<` markert a felhasználónak | e2e | K |
| MRG-10 | **Property:** véletlen párhuzamos szerkesztések → nincs adatvesztés, az eredmény valid | property | K |
| MRG-11 | **Property:** merge után mindig valid frontmatter és valid Markdown | property | K |

---

## 5. Lock / presence (LOCK)

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| LOCK-01 | Edit → lock megszerzése; más user read-only + „X szerkeszti" | integráció | K |
| LOCK-02 | Második Edit ugyanarra a dokumentumra → „foglalt" jelzés | integráció | K |
| LOCK-03 | **Verseny:** két egyidejű Edit-kérés ugyanarra → pontosan egy nyer (atomikus, Redis) | integráció | K |
| LOCK-04 | Lock-lejárat inaktivitás után → automatikus felszabadulás | integráció | N |
| LOCK-05 | Lejárt/elhagyott lock **átvétele** másik user által | integráció | N |
| LOCK-06 | Átvételkor figyelmeztetés, ha mentetlen piszkozat maradna | e2e | N |
| LOCK-07 | Lock felszabadítása Publish után | integráció | N |
| LOCK-08 | Presence: ki nézi a dokumentumot (nem lock, csak jelenlét) | integráció | N |

---

## 6. Piszkozat + retenció (DRAFT)

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| DRAFT-01 | Autosave: minden változás perzisztál a draft-tárba, **nincs commit** | integráció | K |
| DRAFT-02 | A bázis-SHA rögzítése Edit-kor a drafthoz | integráció | K |
| DRAFT-03 | Multi-device: ugyanaz a user másik eszközön folytatja a piszkozatot | integráció | N |
| DRAFT-04 | Az AI a **draftba** ír (ugyanaz az útvonal, nem külön) | integráció | K |
| DRAFT-05 | Retenciós állapotgép: active → dormant (figyelmeztetés) → törölt | unit | N |
| DRAFT-06 | Elhagyott piszkozat törlése a konfigurált idő (~30 nap default) után | integráció | N |
| DRAFT-07 | Felhasználói **elvetés** → azonnali törlés | integráció | N |
| DRAFT-08 | Publish után a draft törlése | integráció | N |
| DRAFT-09 | Tenant-konfigurált retenció felülírja a defaultot | unit | N |
| DRAFT-10 | Piszkozat durability: a szerver újraindítása nem veszíti el a mentett piszkozatot | integráció | K |

---

## 7. GitHub App integráció (GH)

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| GH-01 | Webhook: push a docs-sync-re → mirror `/docs` frissítés + index-rebuild | kontraktus | K |
| GH-02 | Webhook: push a main-re → merge-base frissítés | kontraktus | N |
| GH-03 | Webhook **aláírás-ellenőrzés**: hamis/aláíratlan webhook elutasítása | kontraktus | K |
| GH-04 | Elveszett webhook → a mirror **újraszinkronizálható** (GitHub a forrás) | integráció | K |
| GH-05 | Sparse checkout: **csak** `/docs` materializálódik, **teljes** commit-gráffal | integráció | K |
| GH-06 | Bot push → docs-sync (a bot-identitással) | kontraktus | K |
| GH-07 | PR nyitás (Propose to main): docs-sync → main | kontraktus | N |
| GH-08 | Mirror eldobás + újraklónozás → **azonos** derivált index-állapot | integráció | K |
| GH-09 | Duplikátum-feloldás **git-history-lekérdezéssel** (régebbi commit nyer) | integráció | K |
| GH-10 | Teljes E2E a sandbox-orgon: bekötés → Edit → Publish → PR | e2e | K |

---

## 8. Realtime transport (SSE)

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| SSE-01 | Lock/presence-esemény kézbesítése SSE-n | integráció | N |
| SSE-02 | AI-eredmény **streamelése** SSE-n a draftba | integráció | N |
| SSE-03 | **Reconnect** `Last-Event-ID`-vel: nem vész el esemény | integráció | K |
| SSE-04 | **Polling-fallback** működése, ha az SSE nem elérhető | integráció | K |
| SSE-05 | Redis pub/sub: esemény kézbesítése **több backend-instance** között | integráció | K |
| SSE-06 | Cloud Run request-timeout (≤60 perc) → kliens reconnect, folytonos élmény | e2e | N |
| SSE-07 | **Terhelés:** párhuzamos SSE-session kapacitás mérése (LB/serverless NEG throttling ellenőrzés) | terhelés | K |

---

## 9. Read-forrás + badge (READ)

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| READ-01 | Az olvasónézet a **docs-sync**-ből olvas (nem main) | integráció | K |
| READ-02 | Saját publikálás **azonnal** látszik (nem kell PR-merge) | e2e | K |
| READ-03 | Doksi szinkronban main-nel → **nincs** badge | e2e | N |
| READ-04 | Publikálva, nincs nyitott PR → „friss verzió" badge | e2e | N |
| READ-05 | PR nyitva → „review alatt / javasolt" badge | e2e | N |

---

## 10. Kép + olvasás (IMG)

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| IMG-01 | Dokumentum alapból **read-only** nyílik meg | e2e | N |
| IMG-02 | Képfeltöltés → repo asset-útvonal, Publish-kor commit a Gitbe | integráció | N |
| IMG-03 | Publikálatlan draft-kép a GCS-ben él (nem a Gitben) | integráció | N |

---

## 11. Auth (AUTH)

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| AUTH-01 | GitHub OAuth login → user-identitás (login + email) | integráció | N |
| AUTH-02 | Az email elérhető a `--author` commithoz | integráció | K |
| AUTH-03 | Tenant-izoláció: egy user csak a saját tenant repóit éri el | integráció | K |

---

## 12. Graceful degradation (DEG)

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| DEG-01 | Érvénytelen frontmatter doksi → „⚠ probléma" jelölés, de **nyersen olvasható** | e2e | K |
| DEG-02 | Egy hibás doksi **nem** állítja le az index-rebuildet (a többi doksi rendben) | integráció | K |
| DEG-03 | Duplikált id → **mindkét** doksi látszik, ütközés-jelöléssel | e2e | N |
| DEG-04 | Törött link → „törött" render, a környező szöveg **nem tűnik el** | e2e | N |

---

## Mobil-specifikus manuális kör

Az automatizált tesztek nem fedik a szerkesztő IME/érintőbeviteli viselkedését — ez kézi, valódi-eszközös átnézést igényel.

| ID | Mit ellenőriz | Szint | Pri |
|----|----------------|-------|-----|
| MOB-01 | Szerkesztő IME-bevitel iOS Safari-n (autocorrect, kompozíció) | manuális | K |
| MOB-02 | Szerkesztő IME-bevitel Android Chrome-on | manuális | K |
| MOB-03 | Forrás-mód ↔ WYSIWYG váltás mobilon | manuális | N |
| MOB-04 | Fő hurok Playwright mobil-viewporttal (megnyitás → Edit → Publish → badge) | e2e | K |

---

## CI-besorolás

- **Minden PR-en (gyors, izolált):** minden `unit` és `property` eset; a `testcontainers`-alapú `integráció` esetek.
- **Pre-deploy / ütemezett (lassabb, külső függőség):** a `kontraktus` és sandbox-`e2e` esetek (teszt-org + teszt-App); az `SSE-07` terhelés-mérés.
- **Kézi, kiadás előtt:** a `MOB-*` manuális kör.
- **Coverage-kapu** a kritikus magra: SER, FM, MRG modulok (nem globális százalék).

*(Iterálható: a property-generátorok mélysége; a sandbox-e2e gyakorisága; a konkrét coverage-küszöbök; további élesetek a merge és a szerializáció körül, ahogy valós adat érkezik.)*

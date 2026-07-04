# E09 — Piszkozat-tár + retenció

**Forrás:** koncepció (Szerkesztési modell: Draft→Publish; Adatkezelés és compliance: Retenció; Képek: draft-asset életciklus), technikai terv §2.
**Cél:** durable piszkozat-perzisztencia bázis-SHA-val, multi-device sync, retenciós állapotgép, draft-kép életciklus.
**Függőség:** E04 (`draft` séma), E02 (Postgres, GCS).
**Tesztek:** DRAFT-01…10, IMG-02, IMG-03.

## Döntések (a tervből rögzítve)

- A piszkozat **átmeneti + privát** szerveroldali állapot, úton a Git felé. **Postgresben** (nem Redis) tárolva — a felhasználó munkáját nem veszíthetjük el (durability) + relációs multi-device sync.
- Az **autosave nem commit**: gyakori privát mentés, elválasztva a Publishtól.
- **Bázis-SHA** rögzítése Edit-kor (a Publish 3-way merge-hez, E11).
- **Retenció = adatfajtánként külön**, nem egy TTL. Elhagyott piszkozat állapotgép: `aktív (~7 nap) → szunnyadó (figyelmeztetett) → lejárt → törölt (~30 nap)`. A ~30 nap **tenant-szinten konfigurálható** default.
- **Draft-képek:** GCS-ben (draft-asset), privát átmeneti életciklus; Publish-kor kerülnek a Gitbe, elvetett/lejárt drafttal együtt törlődnek.

## Taskok — Piszkozat

- [ ] **E09-T1** Autosave endpoint: minden változás perzisztál a `draft` táblába, **nincs commit**. **DRAFT-01**.
- [ ] **E09-T2** Bázis-SHA rögzítése Edit-kor a drafthoz (a `docs-sync` HEAD az Edit pillanatában). **DRAFT-02**.
- [ ] **E09-T3** Multi-device sync: ugyanaz a user másik eszközön folytatja ugyanazt a piszkozatot. **DRAFT-03**.
- [ ] **E09-T4** Durability: szerver-újraindítás nem veszíti el a mentett piszkozatot (Postgres-perzisztencia igazolása). **DRAFT-10**.
- [ ] **E09-T5** AI-írás a **draftba** (ugyanaz az útvonal, nem külön) — az interfész itt, az AI-logika E13. **DRAFT-04**.
- [ ] **E09-T6** Új dokumentum draft: `target_path` (nincs `doc_id`/bázis-SHA), a Publish új fájlt commitol (E11-T5-höz kapcsolódik).

## Taskok — Retenció

- [ ] **E09-T7** Retenciós állapotgép: `active → dormant (figyelmeztetés) → törölt`. **DRAFT-05**. (Az utolsó mentéstől ~7 nap aktív, majd szunnyadó.)
- [ ] **E09-T8** Ütemezett törlés a konfigurált idő (~30 nap default) után. **DRAFT-06**.
- [ ] **E09-T9** Tenant-konfigurált retenció felülírja a defaultot (`tenant.retention_config`). **DRAFT-09**.
- [ ] **E09-T10** Felhasználói **elvetés** → azonnali törlés. **DRAFT-07**.
- [ ] **E09-T11** Publish után a draft törlése (E11 hívja). **DRAFT-08**.
- [ ] **E09-T12** Szunnyadó piszkozat figyelmeztetés-csatorna (in-app; e-mail iterálható). „Függő piszkozataim" nézet adatait szolgálja ki (UI E14).

## Taskok — Draft-képek

- [ ] **E09-T13** Képfeltöltés → **draft-asset** a GCS-ben (draft-idhez + userhez kötve), ideiglenes Lore-feloldott hivatkozás a Markdownban. **IMG-03** (a GCS-ben él, nem a Gitben).
- [ ] **E09-T14** Publish-kor a draft-asset stabil repo-útvonalra másolása (`/docs/assets/images/<doc-id>/…`) + a Markdown-hivatkozás átírása + commit (E11 hívja). **IMG-02**.
- [ ] **E09-T15** Draft-asset életciklus: sikeres Publish után törölhető; elvetett/lejárt drafttal együtt törlődik; sikertelen Publishnál megmarad (újrapróbálható).

## Kész-definíció

- Autosave perzisztál commit nélkül, szerver-újraindítás után is megvan (DRAFT-01, DRAFT-10).
- A retenciós állapotgép figyelmeztet, majd a konfigurált idő után töröl (DRAFT-05/06/09).
- Draft-kép a GCS-ben él, Publish-kor kerül Gitbe (IMG-02/03).

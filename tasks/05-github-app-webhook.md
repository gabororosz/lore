# E05 — GitHub App + webhook

**Forrás:** koncepció (Architektúra: üzemeltetési modell, least-privilege; GitHub OAuth + GitHub App modell), technikai terv §1, §4.
**Cél:** GitHub App a bot-identitáshoz, installation-alapú jogosultság, aláírás-ellenőrzött webhook-fogadás.
**Függőség:** E04 (tenant/user), E02 (webhook-végpont Cloud Runon).
**Tesztek:** GH-03 (aláírás-ellenőrzés), GH-06 (bot push), GH-07 (PR nyitás), részben GH-01/GH-02 (webhook → E06).

## Döntések (a tervből rögzítve)

- **GitHub App** adja a bot-identitást (push), az installation-alapú jogosultságot és a webhookokat.
- **Least-privilege jogtípusok:** **Contents** (read/write), **Pull requests** (PR-nyitás), **push webhook**. Nincs Administration/Actions/Secrets.
- Fontos korlát: az App-engedélyek **repo-szinten** hatnak, nem könyvtár-szinten — a `/docs`-only a Lore *saját* önkorlátozása, nem GitHub-kényszer.
- A tenant = a GitHub App installation.

## Taskok

- [ ] **E05-T1** GitHub App regisztráció (least-privilege jogtípusokkal): Contents R/W, Pull requests, push webhook. Prod + teszt-App (E03-T6).
- [ ] **E05-T2** Installation flow: az admin telepíti az Appot a kiválasztott repókra → `tenant` + `repository` rekordok létrejönnek (E04). `docs-sync` branch létrehozása bekötéskor (E06-hoz kapcsolódik).
- [ ] **E05-T3** Installation token kezelése: rövid életű token beszerzése/cache-elése a szerveroldali git- és API-műveletekhez.
- [ ] **E05-T4** Webhook-végpont (Cloud Run): push-események fogadása (`main` és `docs-sync`).
- [ ] **E05-T5** **Webhook aláírás-ellenőrzés** (HMAC): hamis/aláíratlan webhook elutasítása. **GH-03**.
- [ ] **E05-T6** Webhook-esemény → belső job/queue: átadás a mirror-sync + index-rebuild felé (E06/E07). Idempotens feldolgozás (duplikált kézbesítés tűrése).
- [ ] **E05-T7** Bot-identitás a githez: a push a boté, a commit `--author` a szerzőé (a mechanika E11-ben; itt az identitás/token-réteg).
- [ ] **E05-T8** Webhook-fixtúrák rögzítése (record/replay) a kontraktus-tesztekhez (E17): push main/docs-sync payloadok.

## Kész-definíció

- Az App telepíthető egy repóra, létrejön a tenant + repository + `docs-sync` branch.
- Aláírt webhook feldolgozódik, aláíratlan elutasításra kerül (GH-03).
- Rögzített webhook-fixtúrák visszajátszhatók a CI-ben.

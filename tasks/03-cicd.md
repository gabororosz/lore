# E03 — CI/CD pipeline

**Forrás:** technikai terv §4 (CI/CD), §5 (CI-integráció).
**Cél:** automatizált lint/test/build/deploy, kulcs nélküli felhő-auth, teszt-infra a GitHub App-hoz.
**Függőség:** E01, E02.

## Döntések (a tervből rögzítve)

- **Pipeline:** GitHub Actions — lint / test / build → image az Artifact Registrybe → deploy Cloud Runra (+ git-worker).
- **Felhő-auth:** **GitHub OIDC** — nincs hosszú életű felhő-kulcs a CI-ben.
- **Secrets:** Secret Managerből, nem a repóban.
- **Termékspecifikus csavar:** a Lore maga is GitHub App + webhook → dedikált **teszt-GitHub-org + teszt-App** kell, lokálhoz **webhook-forwarding** (smee.io/ngrok).

## Taskok

- [ ] **E03-T1** GitHub Actions alap-workflow: `lint` + `typecheck` + `unit` + `property` + `testcontainers`-integráció **minden PR-en** (gyors, izolált).
- [ ] **E03-T2** Build & push: konténer-image (api, web, git-worker) az Artifact Registrybe.
- [ ] **E03-T3** Deploy: staging automatikus PR-enként (preview), prod külön EU-régióba (védett).
- [ ] **E03-T4** **GitHub OIDC** federáció beállítása (GCP Workload Identity) — nincs statikus service-account kulcs.
- [ ] **E03-T5** Secret Manager integráció a deploy- és runtime-titkokhoz.
- [ ] **E03-T6** **Teszt-GitHub-org + teszt-GitHub-App** provisionálása (dedikált, izolált a prodtól). Lásd E05, E17.
- [ ] **E03-T7** Webhook-forwarding lokál dev-hez (smee.io/ngrok) dokumentálva.
- [ ] **E03-T8** **Pre-deploy / ütemezett** job: GitHub-App kontraktus- és sandbox-e2e tesztek (lassabbak, teszt-orgot igényelnek) + `SSE-07` terhelés-mérés.
- [ ] **E03-T9** **Coverage-kapu** a kritikus magra (SER, FM, MRG modulok) — nem globális százalék.

## Kész-definíció

- PR nyitásra lefut a gyors CI és preview-környezetet deploy-ol.
- Main-re merge prod-deployt indít OIDC-vel (kulcs nélkül).
- A kontraktus/e2e tesztek a teszt-org ellen ütemezetten futnak.

# E02 — Infrastruktúra & IaC (GCP, Terraform)

**Forrás:** technikai terv §3 (infrastruktúra/hosting).
**Cél:** EU-rezidens, multi-tenant SaaS-hosting felállítása Terraformmal, provider-agnosztikusan.
**Függőség:** E01.

## Döntések (a tervből rögzítve)

- **Régió:** GCP `europe-west` (EU-adatrezidencia — compliance-kényszer).
- Tiszta edge/serverless kizárva: a mirror **persistent diszket**, az SSE **hosszú életű kapcsolatot**, a webhook **publikus végpontot**, a rendszer **multi-tenant izolációt** igényel.
- Standard building blockok → egy jövőbeli **EU-szuverén szolgáltatóra** (Scaleway/Hetzner/OVHcloud) váltás reális maradjon.

## Taskok

- [ ] **E02-T1** Terraform váz: state-backend (GCS bucket, europe-west), workspace-ek `staging`/`prod`, provider-agnosztikus modul-struktúra.
- [ ] **E02-T2** **API-tier — Cloud Run** (europe-west): támogatja az SSE streaming-választ; a webhook-végpont is itt fut. Request timeout **felemelve** (SSE-hez, max 60 perc).
- [ ] **E02-T3** **Git-worker — GCE instance + persistent disk** (vagy GKE-pod PVC-vel): valódi git working dir a mirrornak. Az állapot derivált → újraklónozható, nem „szent".
- [ ] **E02-T4** **Cloud SQL for PostgreSQL** (europe-west): tenant/user/index/lock-metaadat/audit + piszkozat-tár.
- [ ] **E02-T5** **Memorystore (Redis)** (europe-west): lock/presence + SSE pub/sub fan-out.
- [ ] **E02-T6** **GCS bucket** (europe-west): draft-képek objektumtára (encryption at rest).
- [ ] **E02-T7** **Artifact Registry** a konténer-image-eknek; **Secret Manager** a titkoknak (nincs titok a repóban).
- [ ] **E02-T8** Hálózat + izoláció: privát kapcsolat az API ↔ Cloud SQL ↔ Memorystore között; publikus csak a webhook/HTTP belépő. Encryption at rest mindenhol.
- [ ] **E02-T9** **SSE-kapacitás validáció (kockázat):** load balancer + serverless NEG throttling ellenőrzése párhuzamos SSE-sessionre (terepi jelentés szerint visszaeshet). Ha szűk → közvetlen Cloud Run URL vagy alternatív LB-konfig. Kötődik: `SSE-07`.
- [ ] **E02-T10** Környezetek: preview/staging + külön EU-régiós prod (a CI innen deploy-ol, E03).

## Kész-definíció

- `terraform apply` felhúzza a teljes staging stacket europe-westben.
- Az api elérhető Cloud Runon, kapcsolódik Cloud SQL-hez és Memorystore-hoz.
- Egy dokumentált terheléses próba igazolja a párhuzamos SSE-kapacitást (vagy rögzíti a fallback szükségességét).

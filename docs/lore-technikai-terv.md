# Lore — Technikai terv

*Kiegészítő dokumentum a Lore projektkoncepcióhoz. A koncepció a „mit és miért", ez a „mivel és hogyan". Minden döntés az MVP-re optimalizált, iterálható, és a koncepcióban rögzített elvekre épül (tárolási filozófia, cserélhető rétegek, EU-adatrezidencia).*

---

## Átfogó elv: cloud-agnoszticitás

Minden réteg cserélhető interfész mögött él — ugyanaz a mintázat, mint a locknál, a transportnál és az LLM-nél. Ok: az insurance-célközönség miatt egy EU-szuverén szolgáltatóra váltás reális jövőbeli követelmény, ezért standard building blockokból (konténer, Postgres, Redis, S3-API, git CLI) építünk, nem menedzselt lock-in primitívekből.

---

## 1. Technológiai stack

| Komponens | Választás | Indok |
|-----------|-----------|-------|
| Frontend | **TypeScript + React**, **PWA** | Hire-elhetőség, ökoszisztéma; PWA a mobile-first célhoz (telepíthető, offline-tűrő piszkozat-cache) |
| Szerkesztő | **TipTap** (ProseMirror-wrapper) | A ProseMirror-döntés pragmatikus megvalósítása; kész az egyedi node-okhoz (`[[wiki-link]]`, image, passthrough) |
| App-backend | **TypeScript/Node — Fastify** | A munka döntően I/O-orkesztráció (GitHub, DB, LLM, SSE); triviális SSE-streaming; közös típusok a frontenddel. Fastify a NestJS helyett: solo fejlesztő, nincs NestJS-rutin — a NestJS kikényszerített szerkezete itt fölös súly, a Fastify vékonysága illik a szokatlan mintákhoz (SSE, git-subprocess) és a framework-független domain-logikához |
| Git-réteg | **valódi `git` CLI** subprocessből | Partial clone (`--filter=blob:none`), sparse-checkout, `git merge-file`. Interfész mögött, hogy később Go-workerbe kiszervezhető legyen |
| Relációs DB | **PostgreSQL** | Tenant/user/index/lock-metaadat/audit; az egyértelmű default |
| Piszkozat-tár | **PostgreSQL** | A felhasználó munkáját nem veszíthetjük el (durability) + relációs multi-device sync — ezért nem Redis |
| Lock + presence + SSE-fanout | **Redis** | TTL-es soft lock natúr Redis-minta; pub/sub fan-outolja az SSE-eseményeket a backend-instance-ok között |
| Keresés | **Postgres FTS** (MVP) | Egy rendszerrel kevesebb; később pluggelhető Meilisearch/Typesense |
| Objektumtár | **GCS** (S3-kompatibilis absztrakció mögött) | Publikálatlan draft-képek; a publikált képek a Gitbe mennek |
| Auth | **GitHub OAuth** | A GitHub App amúgy is központi; azonnal megvan a `--author`-hoz kellő stabil identitás (login + e-mail) |

**Negatív döntés (fontos):** az `isomorphic-git` **nem** alkalmas — nincs sparse/partial clone, nincs valódi merge. A git-réteg szerveroldali, containerbe csomagolt valódi git.

*(Iterálható: a git-réteg későbbi Go-kiszervezése; a keresőmotor cseréje.)*

---

## 2. Adatmodell

A modell a tárolási filozófia három kategóriája mentén tagolódik — ez önmagában dokumentálja, mit kell félteni, mit lehet eldobni, és minek van lejárata.

### Valódi app-állapot (backup-olandó)

- **`tenant`** — a GitHub App installation. `id`, `gh_installation_id`, `retention_config`
- **`user`** — `gh_user_id`, `login`, `email` (a `--author`-hoz), `name`
- **`repository`** — `tenant_id`, `gh_repo_id`, `default_branch`, `docs_path` (default `/docs`), `docs_sync_branch`
- **`audit_event`** — **tartalom-mentes evidencia**: `user_id`, `repo_id`, `doc_id`, `action`, `commit_sha`, `created_at` (ez a GDPR↔AI Act feloldás: metaadat, nem szöveg)
- **`pull_request`** — `repo_id`, `gh_pr_number`, `state` (a badge-hez: van-e nyitott PR)

### Derivált (a mirrorból újraépíthető, eldobható)

- **`document`** — a frontmatter-index egy sora: `repo_id`, `doc_id` (a frontmatter `id`), `current_path`, `title`, `owner`, `status`, `tags`, `docs_sync_sha`
- **`doc_link`** — `from_doc_id`, `to_doc_id` (a `[[wiki-link]]`-gráf éle)
- **`doc_issue`** — az invariáns-validátor eredménye: `repo_id`, `doc_id`/`path`, `issue_type` (missing_id / duplicate_id / invalid_frontmatter / broken_link), `detail`, `detected_at`. Derivált: a mirror-syncnél újraépül
- keresési index (Postgres FTS oszlop/tábla)

### Átmeneti (szerver-tulajdon, TTL-es)

- **`draft`** — `user_id`, `repo_id`, `doc_id` **vagy** `target_path` (új doksinál), `base_sha`, `content`, `created_at`, `updated_at`, `state` (active/dormant), `expires_at`
- **`lock`** — `doc_id`, `holder_user_id`, `acquired_at`, `expires_at` (elsődlegesen Redisben; Postgres csak ha durabilitás kell)
- presence — csak Redis

*(Iterálható: a retention_config granularitása; a document-index normalizálása; a draft képek referenciakezelése publish előtt.)*

---

## 3. Infrastruktúra / hosting

**Régió:** GCP **europe-west** (EU-adatrezidencia).

A koncepció döntései kemény kényszereket adnak, amelyek kizárják a tiszta edge/serverless-t: a mirror persistent diszket akar, az SSE hosszú életű kapcsolatot, és kell publikus webhook-végpont + multi-tenant izoláció. A választott GCP-topológia:

| Réteg | GCP-szolgáltatás | Megjegyzés |
|-------|------------------|-----------|
| API-tier | **Cloud Run** | Támogatja a streaming-választ (SSE); a webhook-végpont is itt |
| Git-worker | **GCE instance + persistent disk** (vagy GKE-pod PVC-vel) | Valódi git working dir kell; a mirror derivált, ezért ez az állapot nem szent (újraklónozható) |
| Relációs DB | **Cloud SQL for PostgreSQL** | europe-west |
| Lock/presence/pubsub | **Memorystore (Redis)** | europe-west |
| Objektumtár | **GCS** | europe-west, draft-képek |
| Registry | **Artifact Registry** | konténer-image-ek |
| Secrets | **Secret Manager** | — |
| IaC | **Terraform** | provider-agnosztikus |

### Cloud Run + SSE — ellenőrzött korlát és következmény

A friss ellenőrzés megerősítette az SSE-döntést, egy kikötéssel, amit be kell tervezni:

- A Cloud Run az SSE-kapcsolatot hosszú életű HTTP-kérésként kezeli, amit a **request timeout** korlátoz: **max 60 perc**, alapértelmezés 5 perc. Ezt fel kell emelni, és a kliensnek **újra kell csatlakoznia** a timeout után — ez az SSE-nél natív (`Last-Event-ID`), tehát illeszkedik. A **polling-fallback** a biztonsági háló.
- A Cloud Run **autoskálázó, állapotmentes** instance-ai miatt az SSE-eseményeket instance-ok között szinkronizálni kell — ezt a **Redis pub/sub** már megoldja (a transport-döntésünk eleve ezt tartalmazza).
- **Kockázat, amit validálni kell:** a Cloud Run egy Global External Application Load Balancer + serverless NEG mögött SSE-nél teljesítmény-visszaeséssel throttle-olhat (terepi jelentés: néhány párhuzamos session a közvetlen Cloud Run URL-hez képest). Terhelés-teszttel ellenőrizni kell a párhuzamos SSE-kapacitást; ha szűk, a közvetlen Cloud Run-végpont vagy alternatív LB-konfiguráció kell. Ez a polling-fallback létjogosultságát is alátámasztja.

### EU-szuverenitás mint jövőbeli út

Az MVP hyperscaleren (GCP) indul — sebesség, ismerős terep —, de a hyperscaler-EU-régió a CLOUD Act miatt reziduális szuverenitás-kockázatot hordoz. Mivel standard building blockokat használunk, egy **EU-szuverén szolgáltatóra** (Scaleway, Hetzner, OVHcloud) váltás reális, ha egy insurance-ügyfél a valódi szuverenitást megköveteli.

*(Iterálható: GCE-worker vs. GKE Autopilot; a Cloud Run párhuzamos-SSE kapacitás valós száma; a szuverén-migráció kiváltó feltétele.)*

---

## 4. CI/CD

- **Pipeline:** GitHub Actions — lint / test / build → konténer-image az Artifact Registrybe → deploy Cloud Runra (+ git-worker).
- **IaC:** Terraform, provider-agnosztikusan.
- **Auth a felhő felé:** **GitHub OIDC** (nincs hosszú életű felhő-kulcs a CI-ben).
- **Secrets:** Secret Managerből, nem a repóban.
- **Környezetek:** preview/staging PR-enként; külön EU-régiós prod.

**Termékspecifikus CI-csavar:** mivel a Lore maga is GitHub App + webhook, a tesztelés dedikált **teszt-GitHub-orgot** és **teszt-GitHub-Appet** igényel, lokális fejlesztéshez **webhook-forwardinggal** (smee.io / ngrok). Ez könnyen alábecsült darab (lásd tesztelési stratégia).

---

## 5. Tesztelési stratégia

A stratégia **kockázat-alapú**: a tesztelési erőfeszítést oda összpontosítjuk, ahol egy hiba **adatvesztést vagy -korrupciót** okoz, nem egyenletesen mindenre. A koncepcióból három ilyen magas kockázatú terület adódik.

### A load-bearing kockázatok (ahol a hibák laknak)

1. **Publish 3-way merge** — a legmagasabb kockázatú logika. Egy hiba itt felülírja vagy elveszíti a felhasználó munkáját.
2. **Markdown szerializáció round-trip** — a minimál-diff nem formázhatja át az érintetlen tartalmat; a passthrough nem veszítheti el az ismeretlen konstrukciókat. Ez közvetlenül védi a „reviewzható, mint a kód" ígéretet.
3. **Frontmatter-invariánsok** — az `id` megőrzése, a figyelmeztető komment túlélése újraszerializáláskor, a wiki-link-feloldás helyessége.

### A teszt-piramis rétegei

**Unit (a legtöbb teszt itt):**
- Markdown-szerializáció: minimál-diff és passthrough — **golden-file** (snapshot) tesztek: adott bemenetre bájtazonos kimenet az érintetlen részeken.
- 3-way merge logika: tiszta eset, konfliktus, elmozdult bázis-SHA.
- Frontmatter parse/serialize: id-megőrzés, komment-megőrzés, duplikátum-detektálás.
- Wiki-link-feloldás az indexből; lock-állapotgép (megszerzés, lejárat, átvétel).
- Retenciós állapotgép (active → dormant → expired).

**Property-based / fuzz (a round-trip és a merge kritikus, ezért nem elég a példateszt):**
- **Round-trip stabilitás:** generálj véletlen, érvényes Markdown-részhalmazt → parse → serialize → az eredmény legyen stabil (idempotens), és az érintetlen részek változatlanok. Ismeretlen konstrukcióra a passthrough bizonyítottan megőrzi a bájtokat.
- **Merge-biztonság:** véletlen párhuzamos szerkesztések → a merge sose veszítsen adatot, és mindig valid frontmatter/dokumentum legyen az eredmény.

**Integráció (valódi függőségek, nem mock):**
- Git-réteg a **valódi `git` CLI** ellen, eldobható temp-repókon (partial clone, sparse, merge, `--author` commit).
- DB + Redis **testcontainers**-szel (valódi Postgres + Redis a CI-ben).

**Kontraktus / komponens (a GitHub App a legkockázatosabb integráció):**
- Webhook-fixtúrák rögzítése és visszajátszása (push a `main`/`docs-sync`-re → mirror-frissítés + index-újraépítés).
- Egy **sandbox teszt-org + teszt-App** a teljes, valódi git-flow E2E-hez (bekötés → edit → publish → PR).

**E2E (a fő hurok):**
- **Playwright**, **mobil viewporttal** és touch-emulációval (mobile-first!): megnyitás → Edit (lock) → szerkesztés → Publish → badge-állapot.
- Konfliktus-forgatókönyv: a barátságos felbontó megjelenése, sosem nyers marker.

### Termékspecifikus teszt-infrastruktúra

- **Teszt-GitHub-org + teszt-GitHub-App**, webhook-forwardinggal (smee.io) lokálhoz.
- **Efemer teszt-repók** tesztfutásonként létrehozva/lebontva (a git-réteg valódi repókon dolgozik).
- **LLM-stub a provider-absztrakció mögött:** a CI **nem** hív valódi LLM-et (költség + nemdeterminizmus) — rögzített fixtúra vagy stub-provider. Valódi LLM csak külön, ritka, jelölt tesztben. A provider-konfiguráció compliance-invariánsait (region-pinnelt EU-végpont, első-féli modellcsalád) automatizált teszt őrzi, nem csak kódreview (lásd: *Kiegészítő kutatás → Google Vertex AI: EU-adatrezidencia és zero-retention/no-training*, valamint a teszteset-katalógus LLM-csoportja).
- **testcontainers** Postgres + Redis az integrációhoz.

### Mobil-specifikus validálás

A szerkesztő IME/érintőbeviteli viselkedése az, ahol a frameworkök leginkább eltérnek — ezt **nem lehet teljesen automatizálni**. Kell egy manuális, valódi-eszközös átnézési kör a szerkesztőre (iOS Safari + Android Chrome), a Playwright-emuláció mellé.

### CI-integráció

- **Minden PR-en:** unit + property-based + integráció (testcontainers). Gyors, izolált.
- **Pre-deploy / ütemezett:** a GitHub-App kontraktus- és E2E-tesztek (ezek a sandbox-orgot igénylik, lassabbak).
- **Coverage-kapu** a kritikus modulokon (merge, szerializáció, frontmatter) — nem globális százalék, hanem a kockázatos magra fókuszálva.

### Amit tudatosan NEM tesztelünk túl

A **derivált index** újraépíthető — a *rebuild* helyességét teszteljük, nem minden egyes lekérdezést. A presence flakiness alacsony tét (nem adatvesztés), ezért nem kap E2E-súlyt.

*(Iterálható: a property-based generátor lefedettségének mélysége; a sandbox-org tesztek gyakorisága; a coverage-küszöbök konkrét számai.)*

---

## 6. Invariáns-robusztusság

A Lore **nem birtokolja a repót**, a dokumentum identitását pedig a frontmatter `id` adja (nem az útvonal), hogy az átnevezés ne törje a `[[wiki-link]]`-eket. Ebből egy törékeny invariáns fakad: **minden doksinak legyen valid, egyedi `id`-je** — de ezt a Lore nem tudja egyedül kikényszeríteni, mert IDE-ből bárki írhat közvetlenül a Gitbe. Az invariáns tehát a Lore hatókörén *kívül* sérülhet.

### A sérülésmódok

1. **Hiányzó `id`** — frontmatter nélküli vagy id nélküli fájl.
2. **Duplikált `id`** — copy-paste, vagy alattomosan egy **branch-merge** (két branch ugyanazt az id-t vezeti be).
3. **Érvénytelen frontmatter** — elrontott YAML vagy sérült `---` határolók.
4. **Törött link** — `[[wiki-link]]` nem létező id-re.

### Vezérelv: detektálás és láthatóvá tétel, nem csendes javítás

A tárolási filozófiából következik: a Lore **nem javíthat automatikusan**, mert minden írás egy attribútált commit (`--author`) + push. Csendes „takarítás" sértené azt az elvet, hogy minden változás szándékos és reviewzható. A Lore ezért *robusztus* egy elromlott repóval szemben (nem omlik össze, nem veszít adatot) és *segít* a javításban — de a javítás mindig ember által jóváhagyott.

### A döntés (MVP-scope)

- **Réteg 1 — Detektálás (MVP).** A webhook-vezérelt mirror-sync + index-újraépítés részeként fut a validátor; a talált problémák a `doc_issue` táblába kerülnek. Nem állítja le a rendszert.
- **Réteg 2 — Graceful degradation (MVP).** Egy elromlott doksi nem rántja le a többit: valid frontmatter nélkül „⚠ probléma" jelöléssel, de nyersen olvasható; duplikált id esetén mindkét doksi látszik, ütközés-jelöléssel; törött link „törött"-ként renderelődik (mint egy 404), a szöveg nem tűnik el.
- **Réteg 4 — Megelőzés a Lore saját írásainál (MVP).** Amikor maga a Lore hoz létre/ment doksit (az esetek többsége), a forrásnál garantálja az invariánst: mindig **egyedi id-t generál** és Publish előtt validálja a frontmattert. Ez a *keletkező* problémák nagy részét megelőzi.
- **Réteg 3 — Vezetett javítás (post-MVP).** A `doc_issue`-khoz egykattintásos, a normál Publish-úton (attribútált commit) menő javítási javaslatok. Tudatosan post-MVP: a Réteg 4 miatt MVP-ben ritka a valódi sérülés, tehát elég, ha a rendszer nem omlik össze és jelez.

### Konkrét szabályok

- **id-formátum: slug + rövid random suffix** (pl. `payment-api-a3f9`). Ütközés-ellenállóbb a tiszta slugnál — pont a branch-merge-eset miatt: két ember sokkal kisebb eséllyel generál azonos random suffixet, mint azonos slugot.
- **Duplikátum-feloldás: a régebbi commit nyer.** Determinisztikus, automatizálható, nincs emberi döntés a hurokban: a `[[wiki-link]]`-ek a korábban bevezetett (régebbi commit) doksira oldódnak fel; az újabb kapja a `duplicate_id` jelölést. (Post-MVP a vezetett javítás felkínálhatja az újabb átnevezését.)

> **Implementációs megjegyzés.** A „régebbi commit nyer" szabály a git-történetből dől el: a validátornak azt kell megállapítania, melyik id **melyik commitban jelent meg először** (`git log` az adott fájlra/tartalomra), nem elég a pillanatnyi fájlállapot vagy az index. Ez a mirror **teljes commit-gráfját** igényli — amit a mirror-döntés (`--filter=blob:none`: teljes gráf, lusta blobok) épp lehetővé tesz. A duplikátum-feloldás tehát history-lekérdezést végez, nem csak indexet olvas; ez az egyetlen validátor-lépés, amelynek a jelenlegi állapoton túl a történetre is szüksége van.

*(Iterálható: a random suffix hossza/ábécéje; a validátor teljessége; a Réteg 3 javítási-UX részletei.)*

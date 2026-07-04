# E10 — Lock/presence + SSE transport

**Forrás:** koncepció (Konkurencia: dokumentum-szintű soft lock; Realtime transport), technikai terv §1 (Redis), §3 (Cloud Run + SSE).
**Cél:** dokumentum-szintű soft lock lejárattal + átvétellel, presence, és a cserélhető SSE-transport (polling-fallbackkel).
**Függőség:** E04 (`lock` séma), E02 (Redis, Cloud Run SSE).
**Tesztek:** LOCK-01…08, SSE-01…07.

## Döntések (a tervből rögzítve)

- **Dokumentum-szintű soft lock** (egyszerre egy szerkesztő), **nem** valós idejű együttszerkesztés. CRDT = post-MVP (nincs itt).
- Lock **soft** (szerver-jelzés, nem Git-zár); **lejárat** inaktivitásra; lejárt/elhagyott lock **átvehető**, de a korábbi draft **nem** hozzáférhető az átvevőnek.
- **Transport: SSE** (egyirányú, ritka, késleltetés-toleráns push); **fallback: polling**; WebSocket csak post-MVP CRDT esetén. **Cserélhető `TransportPort`** mögött (E01-T6).
- **Redis:** TTL-es soft lock natúr minta + **pub/sub** az SSE-események fan-outjára a Cloud Run instance-ok között.
- Cloud Run request-timeout (≤60 perc) → kliens reconnect (`Last-Event-ID`).

## Taskok — Lock

- [ ] **E10-T1** Soft lock megszerzése Edit-kor (Redis, TTL): a holder read/write, mások read-only + „X szerkeszti". **LOCK-01**.
- [ ] **E10-T2** Második Edit foglalt dokumentumra → „foglalt" jelzés. **LOCK-02**.
- [ ] **E10-T3** **Atomikus verseny-feloldás:** két egyidejű Edit ugyanarra → pontosan egy nyer (Redis atomikus művelet). **LOCK-03**.
- [ ] **E10-T4** Lock-lejárat inaktivitás után → automatikus felszabadulás. **LOCK-04**.
- [ ] **E10-T5** Lejárt/elhagyott lock **átvétele** másik user által; a korábbi user piszkozatához **nincs** hozzáférés. **LOCK-05**.
- [ ] **E10-T6** Átvételi figyelmeztetés, ha mentetlen piszkozat maradna. **LOCK-06**.
- [ ] **E10-T7** Lock felszabadítása Publish után (E11 hívja). **LOCK-07**.

## Taskok — Presence

- [ ] **E10-T8** Presence: ki nézi épp a dokumentumot (nem lock, csak jelenlét), avatarok a UI-hoz. **LOCK-08**.

## Taskok — SSE transport

- [ ] **E10-T9** `TransportPort` implementáció SSE-vel: szerver→kliens push a lock/presence-eseményekhez. **SSE-01**.
- [ ] **E10-T10** AI-eredmény streamelése ugyanazon SSE-csatornán a draftba (E13 használja). **SSE-02**.
- [ ] **E10-T11** **Reconnect `Last-Event-ID`-vel:** timeout/hálózatváltás után nem vész el esemény. **SSE-03**.
- [ ] **E10-T12** **Polling-fallback**, ha az SSE nem elérhető (hosting-korlát). **SSE-04**.
- [ ] **E10-T13** **Redis pub/sub fan-out:** esemény kézbesítése több backend-instance között (Cloud Run autoskálázás). **SSE-05**.
- [ ] **E10-T14** Cloud Run timeout (≤60 perc) → kliens reconnect, folytonos élmény. **SSE-06**.
- [ ] **E10-T15** **Terhelés-mérés:** párhuzamos SSE-session kapacitás (LB/serverless NEG throttling). **SSE-07**. Kötődik E02-T9.

## Kész-definíció

- Edit atomikusan foglal; egyidejű verseny esetén egy nyertes (LOCK-03).
- Lock lejár, átvehető; a korábbi draft nem szivárog át (LOCK-05).
- SSE-esemény több instance között kézbesítődik, reconnect nem veszít eseményt (SSE-03/05).
- A polling-fallback működik, ha az SSE tiltott (SSE-04).

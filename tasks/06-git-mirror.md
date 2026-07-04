# E06 — Git-integrációs réteg (sparse `/docs` mirror)

**Forrás:** koncepció (Architektúra: Mirror; gerinc), technikai terv §1 (git-réteg), §3.
**Cél:** per-tenant szerveroldali git-mirror, amely a munkakönyvtárban csak a `/docs`-ot materializálja, de teljes commit-gráffal — a Publish 3-way merge és a duplikátum-feloldás alapja.
**Függőség:** E05 (installation token, webhook).
**Tesztek:** GH-04 (újraszinkron), GH-05 (sparse + teljes gráf), GH-08 (eldobás+újraklón = azonos index).

## Döntések (a tervből rögzítve)

- **Valódi `git` CLI subprocessből** — az `isomorphic-git` **kizárva** (nincs sparse/partial clone, nincs valódi merge).
- **Partial clone (`--filter=blob:none`) + sparse-checkout** a `/docs`-ra: csak a `/docs` materializálódik, blobok lazán — **de a commit-history teljes marad** (a PR merge-base a teljes gráfon dől el).
- A mirror **derivált**: bármikor eldobható és újraszinkronizálható; a GitHub a forrás.
- A git-réteg **`GitPort` interfész mögött** (E01-T6), hogy később Go-workerbe kiszervezhető legyen.

## Taskok

- [ ] **E06-T1** `GitPort` implementáció valódi git CLI-vel, containerizált gittel a git-workeren (E02-T3).
- [ ] **E06-T2** Mirror-provisionálás bekötéskor: partial clone `--filter=blob:none` + sparse-checkout a `docs_path`-ra. **GH-05** (csak `/docs` materializálódik, teljes commit-gráf marad).
- [ ] **E06-T3** Webhook-vezérelt frissítés: push (`main`/`docs-sync`) → `fetch` + a `/docs` sparse-rész frissítése; diff-kimenet átadása az index-rebuildnek (E07).
- [ ] **E06-T4** **Újraszinkronizálhatóság:** elveszett/kimaradt webhook esetén a mirror teljes reszinkronja a GitHubról (GitHub = igazság). **GH-04**.
- [ ] **E06-T5** **Eldobás + újraklónozás** művelet: a mirror teljes újraépítése → **azonos** derivált index-állapot. **GH-08**.
- [ ] **E06-T6** Multi-tenant izoláció a git-workeren: tenantonként külön working dir/repo, tár- és biztonsági szeparáció.
- [ ] **E06-T7** Commit-gráf hozzáférés API-ja a duplikátum-feloldáshoz: „melyik commitban jelent meg először egy adott tartalom/id" (`git log` a fájlra) — E07-T7 használja.
- [ ] **E06-T8** Alacsony szintű git-műveletek a Publishhoz (E11 használja): `fetch`, `merge-file`/3-way merge, `commit --author`, `push`. Interfész-szinten itt, orkesztráció E11.
- [ ] **E06-T9** Hibakezelés: hálózati/GitHub hibák, részleges fetch, disk-nyomás; a mirror sosem hagyható inkonzisztens állapotban (recovery = reszinkron).

## Kész-definíció

- Egy bekötött repóhoz létrejön a sparse `/docs` mirror teljes commit-gráffal (GH-05).
- Push-webhook frissíti a mirror `/docs`-át és triggereli az index-rebuildet.
- A mirror eldobása után az újraépítés bájt-azonos derivált indexet ad (GH-08).

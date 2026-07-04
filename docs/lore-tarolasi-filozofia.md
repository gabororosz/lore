# Tárolási filozófia

> A Git a projekt **publikált** tudásának egyetlen igazságforrása.
> Amit a Lore ezen kívül tárol, az vagy a repóból újraépíthető derivált, vagy egy átmeneti szerkesztési állapot úton a Git felé.
> Ha a Lore megszűnik, a projekt tudása sértetlenül megmarad.

Ez a szekció rögzíti, hogy a Lore-ban **mi él a Gitben, mi él a szerveren, és miért**. Nem stílus­kérdés: ez a döntési keret, amihez minden későbbi „tegyük szerverre" ötletet mérünk, hogy a termék ne csússzon vissza abba a vendor-lock-inbe, amit tagadni akar.

---

## Az alapelv

A Lore **nem** tárol saját, kanonikus dokumentum-adatbázist. A dokumentum hiteles, verziózott, auditálható állapota a Git repositoryban él, a forráskód mellett — így emberek és AI ugyanabból a verziózott tudásból dolgoznak.

Ez az „igazságforrás-ígéret", és ez a termék load-bearing magja: ez adja az auditálhatóságot, a lock-in-mentességet, és azt, hogy a dokumentáció ugyanúgy reviewzható, mint a kód.

Amit viszont **elengedünk**, az a szigorú abszolutizmus („semmilyen adat nem él a Giten kívül"). Ez sosem volt tartható: már az első valódi funkció — keresés, wiki-link-feloldás, AI-kontextus, valós idejű szerkesztés — igényel szerveroldali állapotot. Nem a jobb UX kedvéért adjuk fel; egyszerűen sosem volt igaz.

A kulcs­megkülönböztetés: **a publikálatlan piszkozat nem versenyez a Gittel az igazságforrás szerepéért.** Egy köztes, még-nem-elkötelezett állapot — mint egy el nem mentett Word-dokumentum. Nem sérti az igazságforrás-ígéretet; csak a szigorú változatot.

---

## A három próba

Minden szerveroldali tárolásra vonatkozó döntést ezen a három kérdésen engedünk át. Ha valami mindháromnak megfelel, biztonságos. Ha bármelyiken elbukik, az architekturális figyelmeztető jelzés.

**1. Derivált-e vagy forrás?**
Ha a szerveren tárolt adat bármikor újraépíthető a repóból (keresési index, link-gráf, embeddingek), akkor ártalmatlan: cache, nem igazságforrás, eldobható. Ha viszont *elveszne* a repó törlésével (maga a piszkozat, a kommentszálak), akkor valódi állapot, aminek saját durability-, backup- és GDPR-kezelés kell.

**2. Átmeneti-e vagy végleges?**
A piszkozat *rendeltetése*, hogy eltűnjön — Publish-kor Gitbe olvad, az élettartama rövid. Ez elfogadható. A filozófia valódi eróziója az volna, ha **kanonikus tartalom véglegesen csak a szerveren élne** és soha nem érne Gitbe. Ilyet nem engedünk: az a fajta „gyorsabb, ha ezt csak nálunk tartjuk" döntés az, amitől a projekt fél emlékezete egy nem exportálható adatbázisba csúszik, és visszatér a lock-in.

**3. Mi történik, ha a Lore eltűnik?**
A végső lakmuszpapír. Ha holnap megszűnik a Lore, a felhasználó teljes **publikált** tudása ott van a repójában, sértetlenül, más eszközzel is használhatóan. Csak a félig megírt piszkozatait veszíti el — kellemetlen, de nem katasztrófa, ugyanúgy, ahogy egy el nem mentett dokumentumot is elveszítenél gépösszeomláskor. Amíg ez a mondat igaz marad, a lock-in-mentesség ígérete áll — akármennyi cache és piszkozat ül közben a szerveren.

---

## Mi él hol

| Adat | Hol él | Kategória |
|------|--------|-----------|
| Publikált dokumentumok (Markdown) | Git repository | **Forrás** – igazságforrás |
| Képek, csatolmányok | Git repository (`/docs/assets`) | **Forrás** – igazságforrás |
| Dokumentum-metaadat (frontmatter) | Git repository (a fájlban) | **Forrás** – igazságforrás |
| Commit-történet, PR-ek, review | Git repository | **Forrás** – igazságforrás |
| Keresési index, link-gráf | Szerver | **Derivált** – cache, újraépíthető |
| AI embeddingek / kontextus-index | Szerver | **Derivált** – cache, újraépíthető |
| Publikálatlan piszkozat | Szerver | **Átmeneti** – úton a Git felé |
| Élő szerkesztési munkamenet (ha lesz) | Szerver | **Átmeneti** – úton a Git felé |

A jobb oldali két kategória a szerveren élhet. A bal oldali soha nem élhet *kizárólag* a szerveren.

---

## Következmény: adatkezelés és compliance

Abban a pillanatban, hogy publikálatlan — potenciálisan bizalmas — ügyfél-dokumentumok ülnek a szerveren, a Lore **adatfeldolgozóvá** válik. Ez nem UX-kérdés és nem „majd később" tétel: az enterprise (különösen az insurance-típusú) ügyfél a beszerzési szakaszban fogja kérdezni. Ezért a modell része az elejétől:

- **adatkezelési megállapodás (DPA)** és tiszta felelősség-felosztás;
- **EU-adatrezidencia** a piszkozat- és index-tárolásra;
- **retenciós szabály**: mennyi ideig él egy elhagyott piszkozat, mielőtt automatikusan törlődik;
- **titkosítás nyugalmi állapotban** (encryption at rest) a szerveroldali állapotra;
- annak biztosítása, hogy a **derivált adat bármikor törölhető és újraépíthető** — ez egyben a GDPR-törlési igények kezelését is egyszerűsíti.

---

## Egy mondatban

A Lore a projekt tudását oda helyezi, ahová való: a forráskód mellé, verziózottan, bárhonnan szerkeszthetően, AI-ra felkészítve. A szerver csak gyorsít és közvetít — nem birtokol.

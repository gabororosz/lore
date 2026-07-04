# Lore — Dokumentációs csomag

Ez a csomag a Lore projekt eddig előállt teljes dokumentációját tartalmazza, legfrissebb változatban. A Lore egy Git-native, AI-first tudásréteg szoftverprojektek számára.

## Tartalom és javasolt olvasási sorrend

1. **`lore-projekt-koncepcio.md`** — a fő dokumentum. Executive summary, vízió, filozófia, szerkesztési modell, architektúra, AI, compliance, pozicionálás és a nyitott/eldöntött kérdések listája. A két diagram be van ágyazva (mermaid). *Ha csak egy fájlt olvasol, ez az.*

2. **`lore-technikai-terv.md`** — a „mivel és hogyan": technológiai stack, adatmodell, GCP-hosting, CI/CD, tesztelési stratégia és az invariáns-robusztusság.

3. **`lore-teszt-esetek.md`** — a tesztelési stratégia végrehajtható kibontása: ~70 számozott, kockázat-alapú prioritással ellátott teszteset.

4. **`lore-tarolasi-filozofia.md`** — a tárolási filozófia önálló, README/PRD-kész kifejtése (a koncepcióban is szerepel tömörítve, ez a részletes változat).

5. **`lore-ux-terv.md`** — a „mit lát és mit csinál a felhasználó": a felületek (belépés, projekt-választó, dokumentum-fa, read/edit, új dokumentum, publish + konfliktus-felbontó, PR, AI, keresés, piszkozatok, beállítások), a rajtuk megjelenő információk és funkciók, valamint a konkrét UI-elemek, gombok és mezők.

## Kiegészítő kutatás

- **`lore-vertex-ai-eu-megfeleles.md`** — mélyfúrás a Google Vertex AI EU-adatrezidencia és zero-retention/no-training megfeleléséről: mennyire és milyen feltételekkel elégíti ki az elv-alapú LLM-elhelyezési szabályt. A koncepció *AI: futási mód és LLM-elhelyezés* szekciójához és a technikai terv provider-absztrakciójához ad konkrét végrehajtási részleteket (modellcsalád-korlátozás, végpont-fegyelem, zero-retention mint beszerzési lépés).

## Diagramok (önálló fájlként is)

- **`lore-architektura-diagram.mermaid`** — komponens-diagram (a színek a tárolási filozófiát kódolják: forrás / derivált / átmeneti).
- **`lore-publish-folyamat.mermaid`** — a Publish-folyamat szekvenciadiagramja (Edit → lock → 3-way merge → tiszta/konfliktus ág → Propose-to-main).

Mindkét diagram be van ágyazva a koncepció-dokumentumba is; a `.mermaid` fájlok akkor hasznosak, ha külön (prezentáció, whiteboard, kép-export) kellenek.

## Design system

A vizuális rendszer egy helyen: **`../design-system/`** (repo gyökér).

- **`lore-design-system.html`** — interaktív referencia (színek, tipográfia, komponensek)
- **`lore-design-system.md`** — copy-paste-elhető tokenek és szabályok
- **`lore-mark-*.svg`**, **`lore-icon-tile.svg`** — brand assetek

A mobil prototípus (`../prototype/`) a design tokeneket inline CSS-ben használja; a design system mappára nem hivatkozik fájlútvonallal.

## Státusz

Az MVP-scope-on belül a koncepcionális, architekturális és technikai döntések lezárva, mindegyik a hozzá vezető alternatívákkal és következményekkel dokumentálva (hogy a későbbi iterációk ne nulláról induljanak). Egyetlen tudatosan post-MVP-re halasztott nyitott kérdés maradt: a brown-field migráció (Confluence/Jira/Slack → repo).

A dokumentumok több iteráción át, döntésről döntésre épültek fel; a „döntés + alternatívák + következmények" szerkezet végig szándékos.

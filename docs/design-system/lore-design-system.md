# Lore — Design System

> Git-native, AI-first knowledge layer for software projects.
> Version: v0.1 · License: MIT (open source)

Dokumentáció, ami a kód mellett él — verziózott, mobile-first, AI-first csapatoknak. A rendszer minden felületen őszinte marad arról, **hol él a tudás** és **mennyire lehet benne megbízni**.

---

## Színek

### Szemantikus neonok (storage / lifecycle)

A szín nem dekoráció — megnevezi, *hol* él egy adott tudás és *mennyire megbízható*.

| Név       | HEX       | CSS változó   | Jelentés |
|-----------|-----------|---------------|----------|
| Source    | `#34F5B4` | `--source`    | Publikált igazság. Gitben él — a single source of truth. A Publish akció színe is. |
| Derived   | `#35D6FF` | `--derived`   | Újraépíthető cache — index, linkek, keresés, `/docs` mirror. Nyugodtan eldobható és újraépíthető. |
| Transient | `#FFC24B` | `--transient` | Repülő piszkozatok. A szerveren, úton a Git felé. Rövid életű. |
| Link      | `#B98BFF` | `--link`      | Wiki-linkek, AI és brand. A tudásgráf összekötő szövete. |
| Conflict  | `#FF5C7A` | `--conflict`  | Merge konfliktusok és hibák — az egyetlen pillanat, amikor a Git felszínre tör. |

### Neutrálisok (editor canvas)

| Név        | HEX       | CSS változó   |
|------------|-----------|---------------|
| Ink        | `#090C12` | `--ink`       |
| Surface    | `#11151E` | `--surface`   |
| Surface +  | `#171C28` | `--surface-2` |
| Hairline   | `#242C3A` | `--line`      |
| Line soft  | `#1C2330` | `--line-soft` |
| Text       | `#E8EDF6` | `--text`      |
| Text dim   | `#9AA4B8` | `--text-dim`  |
| Text muted | `#5C6678` | `--text-mute` |

---

## Tipográfia

Három hang, egy rendszer: karakteres display az identitáshoz, csendes workhorse az olvasáshoz, és a monospace mint elsőrangú polgár — mert Lore-ban minden szöveg a Gitben.

| Szerep  | Font          | CSS változó | Fallback stack                  |
|---------|---------------|-------------|---------------------------------|
| Display | Space Grotesk | `--display` | `system-ui, sans-serif`         |
| Body    | IBM Plex Sans | `--body`    | `system-ui, sans-serif`         |
| Mono    | IBM Plex Mono | `--mono`    | `ui-monospace, monospace`       |

**Google Fonts import:**

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=IBM+Plex+Sans:wght@400;500;600&family=IBM+Plex+Mono:wght@400;500&display=swap" rel="stylesheet">
```

**Font súlyok:**
- Space Grotesk: 400, 500, 600, 700
- IBM Plex Sans: 400, 500, 600
- IBM Plex Mono: 400, 500

**Type scale (px):** 72 · 48 · 32 · 24 · 17 (base) · 14 · 12 (mono)

- Base body: `font-size: 1.0625rem` (17px), `line-height: 1.6`
- Display headings: negatív letter-spacing (`-.02em` … `-.04em`), tight line-height (~0.92–1.05)
- Mono label / eyebrow: `.72rem`, `letter-spacing: .16em–.22em`, uppercase

---

## Radius & ritmus

| Token   | Érték    | Használat        |
|---------|----------|------------------|
| `--r-sm`| `6px`    | gombok, chipek   |
| `--r-md`| `10px`   | kártyák, swatch  |
| `--r-lg`| `16px`   | panelek, hero    |
| `--maxw`| `1120px` | tartalom szélesség |

Section padding: `clamp(3.5rem, 8vw, 6rem)`. Wrap padding: `clamp(1.1rem, 4vw, 2.5rem)`.

---

## CSS változók (copy & paste)

```css
:root{
  /* ---- Neutrals: deep-ink editor canvas ---- */
  --ink:        #090C12;
  --surface:    #11151E;
  --surface-2:  #171C28;
  --line:       #242C3A;
  --line-soft:  #1C2330;
  --text:       #E8EDF6;
  --text-dim:   #9AA4B8;
  --text-mute:  #5C6678;

  /* ---- Semantic neons: color that names where knowledge lives ---- */
  --source:     #34F5B4;  /* published truth, lives in Git */
  --derived:    #35D6FF;  /* rebuildable cache */
  --transient:  #FFC24B;  /* drafts in flight */
  --link:       #B98BFF;  /* wiki-links, AI, brand */
  --conflict:   #FF5C7A;  /* merge conflicts, errors */

  /* ---- Type ---- */
  --display: "Space Grotesk", system-ui, sans-serif;
  --body: "IBM Plex Sans", system-ui, sans-serif;
  --mono: "IBM Plex Mono", ui-monospace, monospace;

  /* ---- Radius & rhythm ---- */
  --r-sm: 6px;
  --r-md: 10px;
  --r-lg: 16px;
  --maxw: 1120px;
}
```

---

## Brand mark

- Az **L két találkozó vonal** — pontosan az, amit a Git csinál, amikor két branchet egyesít.
- Egy branch (docs-sync) beleívelódik a fő stroke-ba, és a fényes **source node**-nál áll meg, ahol a single source of truth él.
- A szín a storage rendszert követi: **link → derived → source** (`#B98BFF` → `#35D6FF` → `#34F5B4`).
- Gradient: `linear-gradient` a `#B98BFF`-ből a `#34F5B4`-be.
- **Legibilitás:** a merge-részlet 32px alatt eltűnik — kis méretben csak a tiszta L marad (favicon: 16px, egyszínű `#34F5B4`).
- Monokróm változat sötét és világos háttéren egyaránt működik (`currentColor`).

---

## Komponensek

### Gombok (`.btn`)

`font-weight: 600`, `padding: .62rem 1.15rem`, `border-radius: var(--r-sm)`. Hover: `translateY(-2px)`.

| Variáns        | Class          | Stílus |
|----------------|----------------|--------|
| Alap           | `.btn`         | `--surface-2` háttér, `--line` border |
| Ghost          | `.btn-ghost`   | átlátszó háttér, `--text-dim` szöveg |
| Publish        | `.btn-publish` | zöld (`--source`) — mert a Published állapotot hozza létre |
| Propose        | `.btn-propose` | lila (`--link`) outline |
| Disabled       | `.btn:disabled`| `opacity: .4`, nincs animáció |

### Badge-ek — dokumentum lifecycle szótár (`.badge`)

Pill alak, mono font, `.dot` glow-val (`box-shadow: 0 0 10px currentColor`).

| Állapot                    | Class      | Szín        |
|----------------------------|------------|-------------|
| Synced                     | `.synced`  | `--source`  |
| Published · pending review | `.pending` | `--transient` |
| In review                  | `.review`  | `--link`    |
| Editing — locked           | `.locked`  | `--derived` |
| Draft                      | `.draft`   | `--text-dim`|

### Storage-kategória chipek (`.chip`)

Mono font, kis színes négyzettel: `.source` / `.derived` / `.transient`.

### Egyéb

- **Document card** (`.doc-card`): frontmatter kulcs-érték párok (`.kv`), cím, leírás wiki-linkkel (`.wl`), lábléc chip + badge.
- **Callouts** (`.callout`): `.conflict` (piros) és `.lock` (kék) — irányt mutatnak, nem hangulatot.

---

## Signature: dokumentum lifecycle track

Minden dokumentum ugyanazt a pályát járja be. Minden stage a storage rendszertől kölcsönzi a színét — a paletta és a lifecycle egy gondolat kétszer.

| # | Stage      | Node | Szín        | Leírás |
|---|------------|------|-------------|--------|
| 1 | Read       | R    | text-mute   | Alapértelmezett. Read-only. |
| 2 | Editing    | E    | `--derived` | Egy szerkesztőre zárolva. |
| 3 | Draft      | D    | `--transient` | Autosave-elt, nem committed. |
| 4 | Published  | P    | `--source`  | Committed a docs-sync-re. |
| 5 | In review  | V    | `--link`    | Proposed a main-re. |
| 6 | Merged     | M    | source→link gradient | Reviewed, a main-ben. |

---

## Voice & tone

Egyszerű igék, sentence case, nincs töltelék. Mondd ki mi történik; tartsd meg ugyanazt a nevet a flow során. A hibák magyaráznak és előre mutatnak — nem mentegetőznek.

| ✅ Jó | ❌ Rossz | Miért |
|-------|----------|-------|
| Publish | Submit | Az eredményt nevezd meg, ne a mechanizmust. A Publish gomb Published toastot ad. |
| This document changed while you were editing | Merge conflict at line 42 | A képernyő olvasói oldaláról beszélj. Soha ne exponáld a Git belsőségeit egy nem-technikai szerzőnek. |
| Nothing here yet — start your first doc | No results | Az üres képernyő felhívás cselekvésre, nem zsákutca. |

---

## Egyéb részletek

- **Háttér:** `--ink` (`#090C12`) alapon halvány, fegyelmezett neon `radial-gradient` mező (link + source + derived, alacsony opacitással).
- **Font smoothing:** `-webkit-font-smoothing: antialiased`, `text-rendering: optimizeLegibility`.
- **Focus:** `:focus-visible` → `2px solid var(--link)` outline, `3px` offset.
- **Header:** sticky, `backdrop-filter: blur(12px)`, félig átlátszó ink háttér.
- **Reduced motion:** `@media (prefers-reduced-motion: reduce)` — minden animáció/transition kikapcsolva.
- **Interakció:** bármelyik swatch-re kattintva a hex a vágólapra másolódik.

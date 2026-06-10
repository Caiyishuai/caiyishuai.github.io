# Yishuai Cai — Academic Homepage

> Live site: https://caiyishuai.github.io
> Owner: Yishuai Cai (蔡怡帅), Ph.D. candidate at NUDT, visiting researcher at PKU–PsiBot Joint Lab.

This repository ships a **single-file static homepage**. There is no build step, no
Jekyll, no JS framework — `index.html` contains the full site (HTML + inline CSS +
inline JS) and is served as-is by GitHub Pages.

This README is written so that **future AI agents / LLMs can quickly map a
user-facing request to the exact lines they need to touch**. Use it as the entry
point before opening `index.html`.

---

## 1. Repository Layout

```
.
├── index.html                  # entire site: HTML + inline <style> + inline <script>
├── images/                     # paper figures + avatar (JPEG, ≤800px wide, q=92)
├── images_original/            # untouched 12 MB PNG backups (NOT served)
├── google_scholar_crawler/
│   ├── main.py                 # CI scraper, writes to results/gs_data*.json
│   └── requirements.txt
├── update_citations.py         # local helper: writes google-scholar-stats/gs_data*.json
├── LICENSE
└── README.md                   # ← this file
```

Citation data is **not committed to `main`**; the GitHub Action publishes
`gs_data.json` to the `google-scholar-stats` branch. The page fetches it at runtime
(see §6: Citation Loader).

---

## 2. High-Level Page Architecture

The page is organized as four concentric layers; each later layer assumes the
earlier one is already in DOM.

```
┌────────────────────────────────────────────────────────────────────┐
│ <head>                                                             │
│   • favicon (inline SVG data URI, line ~10)                        │
│   • Font Awesome 6.4.0 CDN (line ~14)                              │
│   • <style> ~1770 lines of CSS (lines 16–1792)                     │
└────────────────────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────────────────────┐
│ <body>                                                             │
│   ├── body::before               top accent ribbon (CSS only)      │
│   ├── .masthead                  sticky top nav        (~1797)     │
│   ├── #main                      grid: sidebar + article (~1844)   │
│   │     ├── .sidebar.sticky      profile card           (~1846)    │
│   │     └── article.page         all content sections   (~1897)    │
│   │           ├── #about-me      .about-bio quote                  │
│   │           ├── #news          <ul> news feed                    │
│   │           ├── #publications  .paper-box × N                    │
│   │           ├── #honors        <ul>                              │
│   │           ├── #education     <ul>                              │
│   │           ├── #experience    .exp-block                        │
│   │           └── #service       .service-block × 2                │
│   ├── footer.site-footer         meta + social         (~2295)     │
│   ├── .back-to-top button                              (~2410)     │
│   └── .lightbox overlay          image zoom            (~2415)     │
└────────────────────────────────────────────────────────────────────┘
┌────────────────────────────────────────────────────────────────────┐
│ Two <script> blocks (~2316–2407, ~2419–2596):                      │
│   1. i18n + BibTeX toggle + Scholar fetch                          │
│   2. lightbox + reveal-on-scroll + scrollspy + theme + footer date │
└────────────────────────────────────────────────────────────────────┘
```

Important global ids referenced from multiple places:

| Selector / id            | Purpose                                | Approx. line |
|--------------------------|----------------------------------------|--------------|
| `#site-nav`              | top nav `<ul class="visible-links">`   | 1800         |
| `#theme-toggle`          | sun/moon button                        | 1829         |
| `.lang-btn`              | 中 / EN switcher                       | 1835–1836    |
| `#about-me … #service`   | scroll-target anchors                  | 1901–2266    |
| `.paper-box`             | publication card (×11)                 | 1917+        |
| `#footer-last-updated`   | auto-filled "Last updated on …"        | 2304         |
| `#back-to-top`           | floating up arrow                      | 2410         |
| `#lightbox` `#lightbox-img` | zoom overlay                        | 2415–2417    |

---

## 3. Design System (CSS Tokens)

All colors / fonts live in **two CSS variable scopes**:

```css
:root          { /* light theme, lines 27–45 */ }
[data-theme="dark"] { /* dark theme overrides, lines 48–64 */ }
```

| Token             | Light value | Dark value | Used for                       |
|-------------------|------------|------------|--------------------------------|
| `--accent`        | `#8b1a1a`  | `#d97373`  | links, hover, ribbon           |
| `--accent-soft`   | `#b03a2e`  | `#e89090`  | secondary highlights           |
| `--accent-deep`   | `#6a1212`  | `#b14444`  | gradients dark stop            |
| `--gold`          | `#c9a44a`  | `#d9b56a`  | featured paper accent          |
| `--ink` / `--ink-soft` / `--muted` | `#1a1a1a` / `#2c2c2c` / `#6b6b6b` | `#ececec` / `#d4d4d4` / `#9a9a9a` | text scale |
| `--bg` / `--bg-soft` / `--bg-tint`  | warm off-white palette | near-black palette | surfaces |
| `--serif`         | Charter / Georgia / Songti SC stack | (same) | body text     |
| `--sans`          | system stack                        | (same) | UI controls   |

**Dark-mode overrides (lines 47–205) follow a strict pattern**: every component
that sets a hard-coded background or color in light mode has a paired
`[data-theme="dark"] .X { ... }` rule. When you add a new component with a
light-baked color, you MUST add a matching dark override here.

Components that already have explicit dark overrides (search anchor: line 47):
`.masthead`, `.lang-switcher`, `.theme-toggle`, `.paper-box.featured`,
`.paper-box:hover`, `.author__avatar img`, `.paper-box-image > div`,
`.site-footer`, `.paper-links a / button.bib-toggle`, `.bibtex-block`,
`.research-chip`, `.conf-card` (+ `.conf-name`, `.conf-year`), `.about-bio`
(+ `::before`, `strong`, `a`, `a:hover`).

---

## 4. Component Reference

Each component is defined twice: **HTML usage** in the body, **styles** in the
`<style>` block. Always edit both when changing semantics.

### 4.1 Masthead (sticky nav)

| Aspect          | Where                                   |
|-----------------|------------------------------------------|
| HTML            | `.masthead` ~line 1797                   |
| Light styles    | `.masthead`, `.masthead.scrolled`, `.masthead::after` ~259, 270, 277 |
| Dark override   | lines 72–79                              |
| Behavior        | `setupMastheadScroll()` ~2472            |

`::after` renders a **scroll progress bar** powered by the CSS variable
`--scroll-progress` set from JS.

### 4.2 Language Switcher (i18n)

| Aspect          | Where                                   |
|-----------------|------------------------------------------|
| HTML buttons    | `.lang-switcher` ~1833                   |
| Function        | `switchLang(lang)` ~2318, `initLang()` ~2344 |
| Mechanism       | every translatable element carries `data-zh` and `data-en` attributes; the function rewrites `textContent` (or `innerHTML` if value contains `<`) on click. |
| Default lang    | hard-coded **English** in `initLang()` (line 2346) — saved preference is intentionally ignored. |

When adding new copy: add `data-zh="..." data-en="..."` to the same element. Keep
HTML inside attributes only when you need it (e.g. `<strong>`, `<a>`); otherwise
use plain text to keep `textContent` path.

### 4.3 Theme Toggle

| Aspect          | Where                                   |
|-----------------|------------------------------------------|
| HTML button     | `#theme-toggle` ~1829                    |
| Icons           | `.theme-icon-dark` (moon, shows in light), `.theme-icon-light` (sun, shows in dark) |
| Function        | `setupTheme()` IIFE ~2554                |
| Persistence key | `localStorage["preferredTheme"] ∈ {light, dark}` |
| First-visit     | follows `prefers-color-scheme`; system changes are tracked **only** while the user has not explicitly toggled. |

### 4.4 Sidebar Profile

| Aspect          | Where                                   |
|-----------------|------------------------------------------|
| HTML            | `.sidebar.sticky` ~1846                  |
| Sub-blocks      | `.author__avatar`, `.affil-list`, `.research-chips` (chips ~1874), `.contact-list` |
| Avatar source   | `images/cys.jpg` (500 px, ~56 KB)        |
| Responsive      | sidebar collapses into row at ≤1024 px (line 1703), into column at ≤768 px (1732). |

### 4.5 About / Bio Quote

| Aspect          | Where                                   |
|-----------------|------------------------------------------|
| HTML            | single `<p class="about-bio">` ~1903     |
| Styles (light)  | lines 797–852                            |
| Styles (dark)   | lines 174–200                            |
| Notes           | Bilingual content stored in `data-zh` / `data-en` attributes (HTML allowed). The big decorative `“` is `.about-bio::before`. |

### 4.6 News List

Plain `<h1 id="news">` + `<ul>` at ~1907. Each `<li>` has an `<em>` date prefix.
Styled by selector `.page__content > #news + ul > li` (line 951) — that selector
is order-sensitive: `<ul>` must immediately follow the `<h1 id="news">`.

### 4.7 Publication Cards (`.paper-box`)

The repeating unit. **11 instances** between ~1917 and ~2232.

```html
<div class="paper-box [featured]">           <!-- .featured = gold left border -->
  <div class="paper-box-image">
    <div>
      <div class="badge">VENUE YEAR</div>
      <img src="images/<id>.jpg" loading="lazy" decoding="async" alt="..." width="100%">
    </div>
  </div>
  <div class="paper-box-text">
    <p><a href="...">Paper title (data-zh / data-en)</a></p>
    <p>Authors (with <strong>self</strong> highlighted; data-zh / data-en)</p>
    <p><span class="honor-tag [honor-oral]">venue tag</span>
       <span class="show_paper_citations" data="<scholar-id>:paperN">| Citations: N</span></p>
    <ul><li>One-line contribution (data-zh / data-en)</li></ul>
    <div class="paper-links">
      <a href="..." target="_blank">Project</a>
      <a href="..." target="_blank">arXiv</a>
      <a href="..." target="_blank">Code</a>
      <button type="button" class="bib-toggle" onclick="toggleBib(this)">BibTeX</button>
      <pre class="bibtex-block">@inproceedings{...}</pre>
    </div>
  </div>
</div>
```

| Aspect              | Where                                   |
|---------------------|------------------------------------------|
| Container styles    | `.paper-box` ~1069, `.paper-box.featured` ~1112 |
| Image / badge       | `.paper-box-image` ~1135, `.badge` (line ~1217) |
| Resource buttons    | `.paper-links` ~1326–1397                |
| BibTeX toggle JS    | `toggleBib(btn)` ~2373; CSS `.bibtex-block.open` (~1383) controls expansion |
| Citation slot       | `.show_paper_citations[data="<author_pub_id>"]` — value currently authored manually; CI does NOT inject into HTML, see §6 |
| Honor tags          | `.honor-tag`, `.honor-tag.honor-oral` ~1475–1508 |
| Image lightbox      | click handler ~2421; opens `#lightbox`   |
| Reveal animation    | added in `setupReveal()` ~2448 (class `.reveal` → `.visible`) |

**Rules for adding a new paper**:
1. Put the figure as `images/<id>.jpg` (≤800 px wide, q≈92).
2. Copy the structure above; mirror `data-zh` / `data-en` on every translatable text node.
3. If venue is NeurIPS / ICML / Spotlight / Oral, add `<span class="honor-tag">…</span>` and optionally `.featured` on the outer `.paper-box`.
4. Use a unique `paperN` suffix in `data="<scholar-id>:paperN"` so the citation loader can target it (currently text is hand-edited; see §6 for the runtime hook).

### 4.8 Honors / Education

Plain `<h1>` + `<ul>` blocks at ~2236 and ~2244, styled by the generic
`.page__content > ul > li` rule (line ~985) which adds the red bullet hover.

### 4.9 Experience (`.exp-block`)

`.exp-block` (~2252) is a **two-column layout**: `.exp-period` left, `.exp-detail`
right. Multiple period/detail pairs may live inside the same `.exp-block`.
Styles ~1400–1473.

### 4.10 Academic Service

Two `.service-block` (~2268, ~2281) each containing:

```html
<h3>Title</h3>
<div class="conference-list">
  <span class="conf-card">
    <span class="conf-name">VENUE</span>
    <span class="conf-year">YEAR(S)</span>
  </span>
  ...
</div>
```

Order of conference cards is significant — currently `NeurIPS → ICML → CVPR → ECCV → AAAI → ICRA → IROS`.

### 4.11 Footer

`<footer class="site-footer">` ~2295. The "Last updated on" date is auto-set from
`document.lastModified` by `setupFooterDate()` IIFE (~2587). Initial fallback
text in HTML is `June 2026`.

### 4.12 Back-to-Top Button

`#back-to-top` (~2410) becomes visible after `scrollY > 320` via
`setupBackToTop()` (~2538). Smooth scroll uses native `window.scrollTo({behavior:'smooth'})`.

### 4.13 Lightbox

`#lightbox` overlay (~2415). Opens by clicking any `.paper-box-image img`,
closes on overlay click or `Escape` key. Locks body scroll while open.

---

## 5. JavaScript IIFE Map

Two `<script>` tags. Functions live at module scope; behavioral blocks are IIFEs
to keep the global namespace clean.

| Block / function          | Lines       | Triggered on                       | Side effects                                   |
|---------------------------|-------------|------------------------------------|------------------------------------------------|
| `switchLang(lang)`        | 2318        | `.lang-btn` click                  | rewrites all `[data-zh]` nodes; sets `localStorage.preferredLang` |
| `initLang()`              | 2344        | DOMContentLoaded (forced English)  | initial render of `[data-zh]` nodes            |
| `toggleBib(btn)`          | 2373        | `.bib-toggle` click                | toggles `.bibtex-block.open`                   |
| Scholar fetch             | 2391–2406   | DOMContentLoaded                   | populates `#total_cit` if present (currently commented out in HTML) |
| Lightbox                  | 2421–2443   | image click + Esc                  | toggles `#lightbox.active`                     |
| `setupReveal()`           | 2448        | once at parse                      | IntersectionObserver adds `.visible` to `.paper-box`, `.exp-block`, `.service-block`, `h1` |
| `setupMastheadScroll()`   | 2472        | scroll, resize                     | toggles `.masthead.scrolled`, sets `--scroll-progress` |
| `setupScrollSpy()`        | 2496        | scroll                             | toggles `.is-active` on nav links              |
| `setupBackToTop()`        | 2538        | scroll                             | toggles `.back-to-top.visible`                 |
| `setupTheme()`            | 2554        | parse + click + system change      | sets `data-theme` attr; persists `preferredTheme` |
| `setupFooterDate()`       | 2587        | once at parse                      | writes month + year into `#footer-last-updated` |

All IIFEs are **idempotent and self-guarding** (`if (!el) return;`). Safe to
reorder, but keep `setupTheme` early so the theme is applied before paint.

---

## 6. Citation Loader (Google Scholar)

Two parallel paths exist; only the runtime fetch actually affects the rendered page:

1. **CI scraper** — `google_scholar_crawler/main.py`
   Run by GitHub Actions, writes `results/gs_data.json` and pushes to the
   `google-scholar-stats` branch. Requires `GOOGLE_SCHOLAR_ID` env var
   (currently `woOucrYAAAAJ`).

2. **Local helper** — `update_citations.py`
   Same logic, writes to `./google-scholar-stats/`. Useful for debugging.
   ```bash
   pip3 install scholarly==1.5.1
   python3 update_citations.py
   ```

3. **Runtime fetch** — inline JS at `index.html:2391`
   ```js
   fetch('https://raw.githubusercontent.com/caiyishuai/caiyishuai.github.io/google-scholar-stats/gs_data.json')
     .then(r => r.json())
     .then(data => { /* writes data.citedby into #total_cit */ });
   ```
   **Currently `#total_cit` is commented out** in HTML (line 1905). To re-enable
   the total citations badge, uncomment that paragraph; per-paper counts
   (`.show_paper_citations[data="...:paperN"]`) are still authored manually.

---

## 7. Image Pipeline & Performance

All paper figures and the avatar were optimized once with macOS `sips`:

```bash
# resize (≤800px for figures, ≤500px for avatar)
sips -Z 800 <fig>.png --out <fig>.png
sips -Z 500 cys.png --out cys.png
# convert to JPEG q=92 (alpha channels were inspected and found unused)
sips -s format jpeg -s formatOptions 92 <fig>.png --out <fig>.jpg
```

Result: **12 MB → 1.9 MB**. Originals are kept in `images_original/` (not
referenced by HTML). Every figure `<img>` carries `loading="lazy" decoding="async"`;
avatar (`images/cys.jpg`) is **not lazy** because it is above the fold.

When adding a new figure, follow the same recipe and add the `loading="lazy"
decoding="async"` attributes manually.

---

## 8. Local Preview & Deployment

```bash
# any static server works
python3 -m http.server 4000
# → http://127.0.0.1:4000
```

Deployment: push to `main`. GitHub Pages serves `index.html` directly.
The `google-scholar-stats` branch is auto-managed by the citation Action.

---

## 9. Editing Cheat Sheet for Agents

| Task                                            | Touch                                                           |
|-------------------------------------------------|-----------------------------------------------------------------|
| Add new publication                             | duplicate a `.paper-box` block (~1917–2232); add image to `images/` |
| Add news item                                   | new `<li>` inside `<ul>` after `#news` (~1907)                  |
| Change navigation order / add section           | edit `.visible-links` (1801) **and** add matching `<h1 id="...">` in article (~1900+) — scrollspy auto-discovers from href |
| Add a translatable string                       | add `data-zh="..." data-en="..."` to the element                |
| Add a new component with hard-coded color       | also add `[data-theme="dark"] .X { ... }` near line 47–200      |
| Adjust theme palette                            | edit `:root` (27) and `[data-theme="dark"]` (48) variables only |
| Reorder service / conference cards              | edit `.conference-list` blocks at 2270 and 2283                 |
| Change "Last updated"                           | nothing — it's auto from `document.lastModified`                |
| Disable / re-enable total citation count        | uncomment line 1905, ensure `gs_data.json` reachable on `google-scholar-stats` branch |
| Tune lazy-load / image quality                  | re-run `sips` recipe in §7; keep new files under `images/`      |

When in doubt, search by **section anchor id** (`#about-me`, `#publications`, etc.)
or **component class** (`.paper-box`, `.conf-card`, `.about-bio`); these are the
two most stable indexes into the file.

---

## 10. Acknowledgements

Originally adapted from
[RayeRen/acad-homepage.github.io](https://github.com/RayeRen/acad-homepage.github.io),
later rewritten as a self-contained single HTML page.
Content is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

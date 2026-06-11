# Portfolio — Project Context (Auto-loaded by Claude)

This is Weiqi Shi's **personal portfolio website**. It is a completely standalone static site with **no connection to any other project** (including Engram).

---

## What This Project Is

A single-file static portfolio site for Louis Shi (Weiqi Shi) — builder, product manager, fintech/Web3 operator. Deployed on Cloudflare Pages via GitHub auto-deploy.

**No backend. No database. No build step.** Everything lives in one file: `index.html`.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML + CSS + JS (single file) |
| Hosting | Cloudflare Pages |
| Version control | GitHub → Cloudflare Pages auto-deploy |
| Font | DM Serif Display (Google Fonts) + Tabler Icons |

---

## Live URLs

| Service | URL |
|---------|-----|
| Production | https://louisshi-portfolio.pages.dev |
| GitHub | https://github.com/Supremelouis/louis-shi-portfolio (`master` branch) |

---

## Directory Structure

```
Portfolio/
├── CLAUDE.md          # This file — auto-read by Claude Code
├── index.html         # The entire site — all HTML, CSS, and JS in one file
├── hero-photo.jpg     # Hero portrait photo (also used for nav avatar + OG image)
├── imgs/              # Case-study carousel background images
│   ├── case-pathport.jpg
│   ├── case-harena.jpg
│   └── case-engram.jpg
└── .claude/
    └── launch.json    # Preview server config (npx http-server, port 5500)
```

**Asset hygiene:** every image in this repo should be referenced from `index.html`
(check `CS_IMGS`, `<img src=...>`, `background-image:url(...)`). Remove unused
images when they're replaced — earlier iterations left ~40 dead logo/timeline
images in `logos/` and `imgs/exp*` that were cleaned up on 2026-06-10. The
nav avatar and About modal previously used `portrait.jpg`, which was removed
on 2026-06-10 (both now reference `hero-photo.jpg`, and the About modal no
longer has a portrait button).

---

## Common Commands

### Local Preview

```bash
# Start local server (from Portfolio directory)
npx http-server -p 5500 -c-1
# Then open http://localhost:5500
```

The `.claude/launch.json` configures this server for the Preview MCP tool.
(Python is not available on this machine — use `npx http-server`, not `python -m http.server`.)

### Deployment

```bash
# Push to GitHub → Cloudflare Pages auto-deploys (1-2 min)
git add index.html
git commit -m "feat/fix: description"
git push origin master
```

**Workflow:** Always edit locally → preview → user approves → then push. Never push without user confirmation.

---

## Architecture of index.html

The file has three logical sections in order:

1. **`<head>`** — meta, Google Fonts, Tabler Icons CDN
2. **`<style>`** — all CSS (minified, one rule per line)
3. **`<body>`** — HTML structure, then About Modal, then `<script>`

### Critical DOM ordering rule

The About Modal HTML **must appear BEFORE** the `<script>` block. The script calls `setLang('en')` on page load which runs `getElementById('ab-p1')` etc. — if modal HTML is after `</script>`, those elements don't exist yet and the modal renders empty.

### Bilingual system

All visible text is driven by a data object in the `<script>`:

```js
const D = {
  en: { hTitle: '...', hSub: '...', /* etc */ },
  zh: { hTitle: '...', hSub: '...', /* etc */ }
}
function setLang(lang) { /* calls setText(id, html) for every element */ }
```

The language toggle in the nav switches between `setLang('en')` and `setLang('zh')`.

---

## Hero Section Layout

The hero uses **CSS Grid** with two columns:

```css
.hero {
  display: grid;
  grid-template-columns: 1fr 1.02fr;
  gap: 48px;
  align-items: stretch;    /* both cells same height */
  align-content: center;   /* grid centered in min-height:100vh */
  position: relative;      /* needed for scroll-hint absolute positioning */
}
.hero-text {
  display: flex;
  flex-direction: column;
  justify-content: center; /* text centered vertically in cell */
}
.portrait {
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%); /* portrait centered vertically in cell */
}
```

**Critical:** `align-content:center` must stay on `.hero` or the single auto row expands to full viewport height, pushing all sections below the fold.

### Column widths at max-width (1180px viewport+)

- Left column: ~542px
- Right column: ~554px
- Portrait width: 340px → portrait left edge at 554–340 = 214px from right-column start

### Hero title font size constraint

Title line 2 is "depending on where you're standing." (~35 chars). At ~0.47em average char width:
- **32px max** → line width ≈ 527px < 542px ✓ (safe)
- **34px** → line width ≈ 560px > 542px ✗ (wraps to 3 lines at 1366px)

**Do not raise `clamp` max above 32px** without either shortening the title text or widening the left column.

---

## CSS Custom Properties (`:root`)

Key variables used throughout:

| Variable | Usage |
|----------|-------|
| `--txt` | Primary text (dark) |
| `--txt2` | Secondary text (medium) |
| `--txt3` | Tertiary text (faint) |
| `--ac` | Accent color (blue) |
| `--glass` | Glass card background |
| `--gb` | Glass card border |
| `--sh` | Standard box shadow |
| `--sh-h` | Heavier box shadow (portrait) |
| `--line` | Divider line color |

**Pitfall:** `--card` and `--cb` are NOT defined in `:root`. Using them produces transparent backgrounds. Use `--glass` / `--gb` instead.

---

## Mobile Breakpoint

```css
@media (max-width: 920px) {
  .hero { grid-template-columns: 1fr; text-align: center; }
  .portrait { top: 50%; left: 50%; right: auto; transform: translate(-50%, -50%); }
}
```

At ≤920px: single column, portrait centered absolutely within hero-visual.

---

## Known Pitfalls

1. **About modal empty** — Modal HTML must be BEFORE `<script>`. Moving it after causes `getElementById` to return null on page load.

2. **Writing section no background** — `.writ-list` must use `var(--glass)` / `var(--gb)`, not `var(--card)` / `var(--cb)` (undefined).

3. **scroll-hint at wrong position** — `.hero` needs `position:relative` or the `position:absolute; bottom:18px` scroll hint anchors to `.wrap` instead of the hero section.

4. **3-line title wrap** — Raising hero-title font max above 32px causes line 2 to wrap at 1366px viewport width (a common laptop resolution).

5. **Mobile portrait broken by desktop top:0** — If desktop `.portrait` uses `top:0; transform:none`, the mobile override must explicitly set `top:50%` (don't rely on cascade).

6. **Multiple JSX/HTML root elements** — Not applicable (vanilla HTML), but if converting: multiple sibling roots need a Fragment wrapper.

---

## Sections (in DOM order)

1. **Nav** — fixed top bar with logo, links, language toggle, theme toggle
2. **Hero** — two-column grid: left = text + CTA buttons, right = portrait + floating cards
3. **Work** — project case studies
4. **Timeline** — career history
5. **Writing** — articles/essays list
6. **Career** — detailed experience
7. **Contact** — footer with links
8. **About Modal** — overlay, opened via "About Me →" button in hero-actions

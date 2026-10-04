# Organic-Shapes-Generative-Art

A seed-based generative art catalogue for **Organic Shapes**, a computational petal-form system. Built as a single-file HTML page with p5.js, styled as a gallery-ready portfolio and licensing surface.

**Live:** `https://reyrove.github.io/Organic-Shapes-Generative-Art/`

---

## Overview

**Organic Shapes** renders a 6 × 6 grid of layered organic forms from numeric seeds. Each form is drawn from a flowing parametric formula — a base radius modulated by sine and cosine waves at 3× and 5× frequency — then rendered in three concentric strokes at descending brightness and alpha. Five curated palettes (Ice, Fire, Strawberry, Forest, Sunset) drive the colour choices, with one palette picked per seed.

The page functions as a **catalogue and licensing surface** for fashion houses, textile studios, and surface designers. Every composition is reproducible by seed, meaning the same number always produces the identical field.

---

## Features

### Generative Engine (p5.js)

- **6 × 6 grid** — 36 organic forms per composition, each centred in its cell
- **Parametric petal formula** — each form's radius is a sum of two sine/cosine modulations, sampled at 120 points:
  ```js
  r = s * (0.4
    + 0.2 * sin(3θ + layer * 0.5)
    + 0.15 * cos(5θ - layer * 0.3));
  ```
- **Three concentric strokes per form** — layer 0 at full brightness and 60% alpha, layer 1 at −10 brightness and 45% alpha, layer 2 at −20 brightness and 30% alpha — producing a soft nested contour
- **Five curated palettes**:
  - **Ice** — `#A3D9FF`, `#C7F0FF`, `#E0FFFF`, `#B0E0E6`
  - **Fire** — `#FF4500`, `#FF6347`, `#FFA500`, `#FFD700`
  - **Strawberry** — `#FF6B81`, `#FF99AA`, `#FFB3C6`, `#FFE0E5`
  - **Forest** — `#2E8B57`, `#3CB371`, `#66CDAA`, `#98FB98`
  - **Sunset** — `#FF7F50`, `#FF6347`, `#FF4500`, `#FFA07A`
- **Per-form variation** — each form has its own size (0.5–0.9 of cell), rotation (0–2π), and colour from the palette
- **Deep blue-violet ground** — a background drawn from hue 200–260, saturation 15–40, brightness 10–25

### Same Seed → Many Surfaces

A single seed drives every instance on the page:

| Instance | Purpose | Tiling |
|----------|---------|--------|
| **Cover** | Hero composition | 1 × 1 |
| **Plate** | Main work (master) | 1 × 1 |
| **Frame** | Framed archival print | 1 × 1 |
| **Print** | Archival print surface | 1 × 1 |
| **Scarf** | Silk scarf (3:1) | 3 × 1 |
| **Textile** | Fabric yardage (4:3) | 2 × 2 |
| **Wall** | Wallpaper (2:3) | 1 × 2 |
| **Archive** | 8 unique seeds | 1 × 1 each |

### Controls

- **New Seed** button / `R` key — regenerate with a fresh seed
- **Download** button / `S` key — export the plate canvas as PNG
- **Click any archive plate** — load that seed into the main view

### Performance

- Debounced resize handling (250ms)
- `IntersectionObserver` pauses off-screen sketches
- `visibilitychange` pauses rendering when the tab is hidden
- `pixelDensity(1)` for a lighter canvas footprint
- Two-frame render (`maxFrames: 2`) so tiles draw once and stop

---

## File Structure

```
Organic-Shapes-Generative-Art/
├── index.html          # Single-file page (structure + styles + generative engine)
├── README.md
└── images/
    ├── fav.svg         # Favicon
    ├── tote.png        # Mockup: tote bag
    ├── tee.png         # Mockup: t-shirt
    └── cushion.png     # Mockup: cushion
```

No build step. No dependencies to install. p5.js is loaded from a CDN.

---

## Getting Started

1. Clone the repo:
   ```bash
   git clone https://github.com/reyrove/Organic-Shapes-Generative-Art.git
   cd Organic-Shapes-Generative-Art
   ```
2. Add the three mockup images to `images/` (optional — they appear in the Commission section).
3. Open `index.html` in a browser.

That's it. The page self-initialises on `DOMContentLoaded`.

---

## How the Generative System Works

### Seeded Random

A linear congruential generator replaces p5's native randomness inside each sketch:

```js
class SeededRandom {
  constructor(s) { this.state = (Math.abs(s) % 2147483647) || 1; }
  rand() { this.state = (this.state * 9301 + 49297) % 233280; return this.state / 233280; }
  range(min, max) { return this.rand() * (max - min) + min; }
  int(min, max) { return Math.floor(this.range(min, max + 1)); }
  pick(arr) { return arr[Math.floor(this.rand() * arr.length)]; }
}
```

Every aesthetic decision — background hue, theme selection, per-form size, rotation, colour — is drawn from this generator in a fixed order. Same seed, same sequence, same composition.

### Parameter Space (per seed)

| Parameter | Range |
|-----------|-------|
| Grid size | 6 × 6 |
| Background hue | 200 – 260 |
| Background saturation | 15 – 40 |
| Background brightness | 10 – 25 |
| Theme | 5 options (Ice → Sunset) |
| Palette size | 4 colours per theme |
| Form size | 0.5 – 0.9 × cell |
| Form rotation | 0 – 2π |
| Sampling points | 120 per form |

### The Petal Formula

The radius of each vertex is computed as a sum of two modulations on a base:

```js
r = s * (0.4
  + 0.2 * sin(3θ + layer * 0.5)
  + 0.15 * cos(5θ - layer * 0.3));
```

The `sin(3θ)` term produces three broad lobes, and the `cos(5θ)` term produces five shallower notches. Together they make a form with three petals and five soft creases — organic, never a perfect circle. The `layer` offset (`layer * 0.5` and `layer * 0.3`) shifts the modulation slightly between the three concentric strokes, so the layered outlines don't nest perfectly — they drift, giving the form a subtle inner motion.

### The Layered Strokes

Every form is drawn three times:

```js
for (let layer = 0; layer < layers; layer++) {
  p.stroke(
    h,
    sat,
    Math.max(0, bri - layer * 10),      // descending brightness
    Math.max(0, 0.6 - layer * 0.15)     // descending alpha
  );
  // draw the same closed shape
}
```

Layer 0 draws at full brightness and 60% alpha. Layer 1 draws at −10 brightness and 45% alpha. Layer 2 draws at −20 brightness and 30% alpha. The result reads as three nested outlines: a bright outermost contour, a mid-tone middle contour, and a faint innermost contour. Together they give the form depth without needing a fill.

### Colour Handling

Themes are stored as hex strings but converted to HSB triples for consistency with the rest of the catalogue:

```js
function hexToHSB(hex) {
  const r = parseInt(hex.slice(1, 3), 16) / 255;
  const g = parseInt(hex.slice(3, 5), 16) / 255;
  const b = parseInt(hex.slice(5, 7), 16) / 255;
  // ... convert to [h, s, v]
}
```

Each of the four palette colours becomes an `[h, s, b]` triple that can be passed directly to `p.stroke()`.

### Tiling Logic

Rectangular surfaces (scarf, textile, wall) tile the square composition N×M. Each tile is drawn into its own translated coordinate space, centred within its cell:

```js
const tileW = canvasW / tileCols;
const tileH = canvasH / tileRows;
const tileSize = Math.min(tileW, tileH);
// draw square of side tileSize at (ox, oy) for each cell
```

This preserves the square composition's proportions regardless of surface aspect ratio.

---

## Customisation

### Change the default seed

Edit the init function:

```js
function init() {
  seed = 123456789;  // fixed seed instead of random
  // ...
}
```

### Adjust the grid resolution

Edit the constants near the top of the script:

```js
const COLS = 6;
const ROWS = 6;
```

Higher values (8, 10) produce denser grids of smaller forms. Lower values (4, 5) produce larger, more legible individual shapes.

### Add a new theme

Add a new entry to `THEMES`:

```js
const THEMES = [
  { name: 'Ice',        colors: ['#A3D9FF', '#C7F0FF', '#E0FFFF', '#B0E0E6'] },
  { name: 'YourTheme',  colors: ['#...', '#...', '#...', '#...'] },
  // ...
];
```

The seed will automatically pick it up on the next regeneration.

### Change the petal formula

Inside `drawOrganicShape`:

```js
const r = s * (
  0.4 + 0.2 * Math.sin(3 * theta + layer * 0.5)
      + 0.15 * Math.cos(5 * theta - layer * 0.3)
);
```

- **Change `3` and `5`** for different lobe and notch counts (e.g., `4` and `7` for a more complex form).
- **Change `0.2` and `0.15`** for stronger or weaker modulation.
- **Change `0.4`** for a larger or smaller base radius.
- **Add a third term** (e.g., `+ 0.1 * Math.sin(8 * theta)`) for a more complex silhouette.

### Adjust the layered strokes

```js
const layers = 3;
```

Increase for more contours (each one dimmer and more transparent). Decrease to 1 for a single hard outline.

### Adjust the stroke weight

```js
p.strokeWeight(cs * 0.005);
```

Change the multiplier for a heavier or lighter outline. The current ratio (0.5% of canvas) reads as a substantial pen.

### Change the background

```js
const bgHue = sr.range(200, 260);
const bgSat = sr.range(15, 40);
const bgBri = sr.range(10, 25);
```

Replace with fixed values for a consistent ground across all seeds, or widen the ranges for more variance.

### Change surface tiling

Edit `surfaceDefs`:

```js
const surfaceDefs = [
  { id: 'surfacePrint',   wrapId: 'wrap-surfacePrint',   tile: { cols: 1, rows: 1 } },
  { id: 'surfaceScarf',   wrapId: 'wrap-surfaceScarf',   tile: { cols: 3, rows: 1 } },
  // ...
];
```

The `tile` object accepts any `{ cols, rows }` pair.

### Replace mockup images

Swap the files in `images/` or update the `src` attributes in the Commission section:

```html
<img src="images/your-mockup.png" alt="..." loading="lazy" />
```

---

## Design System

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#0a0a0a` | Page background |
| `--paper` | `#f4f1ea` | Light surface (Surfaces section) |
| `--gold` | `#b8935a` | Primary accent |
| `--gold-soft` | `#d4b483` | Italic emphasis |
| `--serif` | Cormorant Garamond | Headings, titles |
| `--mono` | DM Mono | Labels, metadata, UI |

Type scale, spacing, and section rhythm match the Skyline, Portrait, Girih, Gothic Grid, Order to Chaos, Wallpaper Groups, Wandering Tiles, Food Grid, Framed Line, Deconstructed Stage, Layered Diagram, Japanese Patterns, and Matrix Rain templates. Responsive breakpoints at 900px, 720px, 560px, and 400px.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Regenerate with a new seed |
| `S` | Download plate as PNG |
| `Enter` / `Space` | Activate focused archive item |
| `Esc` | Close legal modal |

---

## Browser Support

Modern evergreen browsers with:

- Canvas 2D (universal)
- `IntersectionObserver` (universal in modern browsers)
- `backdrop-filter` (Safari 9+, Chrome 76+, Firefox 103+)
- CSS custom properties (universal)

Fallback: if `IntersectionObserver` is unavailable, sketches render continuously.

---

## Accessibility

- Archive items are keyboard-focusable with `role="button"` and `aria-label`
- Legal modal traps focus, closes on `Esc` and overlay click
- `prefers-reduced-motion` disables transitions and smooth scroll
- All interactive elements have visible focus states (via browser default)

---

## Section Map

| Section | ID | Purpose |
|---------|-----|---------|
| Cover | — | Hero title + live field |
| Statement | `#statement` | Artist statement |
| Plate | `#work` | Main work + metadata + controls |
| Surfaces | `#application` | Framed print + 4 tiled surfaces |
| Process | `#process` | Three process cards |
| Archive | `#archive` | 8 selectable seeds |
| Commission | `#commission` | Services + mockups + CTA |
| Colophon | — | Footer, links, legal |
| Legal modal | `#legalModal` | Licensing / Terms / Credits |

---

## Deployment (GitHub Pages)

1. Push the repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, select `Deploy from a branch`.
4. Choose branch `main` and folder `/ (root)`.
5. Save — the site goes live at `https://reyrove.github.io/Organic-Shapes-Generative-Art/`.

---

## Notes on the Source Sketch

The generative engine is adapted almost verbatim from the original p5 sketch. The adaptations that turn it into a catalogue page:

1. **Seeded random replaces p5's `random()`** — a local `SeededRandom` class drives every choice. The original sketch calls `setSeeds()` for fxhash compatibility; that's replaced with the seeded generator directly.
2. **Sketch factory** — `makeSketch({ slotId, seedValue, ... })` returns a p5 sketch that can be instantiated into any DOM slot, enabling cover / plate / frame / surfaces / archive from one seed.
3. **DOM-driven canvas sizing** — `cs = min(windowWidth, windowHeight)` becomes slot-rect dimensions.
4. **Tiling support** — the `tile: { cols, rows }` parameter repeats the square composition across rectangular surfaces.
5. **Drawing logic takes `p` explicitly** — the original used p5's global mode; here `p` is passed so the factory closure works.
6. **Hex → HSB conversion inlined** — the original uses p5's `color('#hex')` and reads `hue()`, `saturation()`, `brightness()`. Inside the factory, this is replaced with a `hexToHSB()` helper that converts hex strings to `[h, s, b]` triples directly.
7. **`map(i, 0, points, 0, TWO_PI)` inlined** — replaced with `(i / points) * Math.PI * 2`.
8. **`windowResized` replaced by a debounced `handleResize`** — all sketches are destroyed and recreated on resize (250ms debounce), since their canvases are locked to slot dimensions.
9. **`triggerPreview` removed** — the original calls it for fxhash thumbnail generation; not needed in the plain-browser catalogue.

---

## Related Repositories

| Repo | Theme |
|------|-------|
| [`Skyline-Series`](https://github.com/reyrove/Skyline-Series) | Themed cityscapes |
| [`Portrait-Series-Generative-Art`](https://github.com/reyrove/Portrait-Series-Generative-Art) | Generative faces |
| [`Girih-2-Generative-Art`](https://github.com/reyrove/Girih-2-Generative-Art) | Islamic geometric ornament |
| [`Gothic-Grid-Generative-Art`](https://github.com/reyrove/Gothic-Grid-Generative-Art) | Mosaic grid composition |
| [`Order-to-Chaos-Generative-Art`](https://github.com/reyrove/Order-to-Chaos-Generative-Art) | Gradient grid composition |
| [`Wallpaper-Groups-Generative-Art`](https://github.com/reyrove/Wallpaper-Groups-Generative-Art) | Crystallographic symmetry |
| [`Wandering-Tiles-Generative-Art`](https://github.com/reyrove/Wandering-Tiles-Generative-Art) | Line-drawing composition |
| [`Food-Grid-Generative-Art`](https://github.com/reyrove/Food-Grid-Generative-Art) | Illustrated food grid |
| [`Framed-Line-Generative-Art`](https://github.com/reyrove/Framed-Line-Generative-Art) | Gesture line composition |
| [`Deconstructed-Stage-Generative-Art`](https://github.com/reyrove/Deconstructed-Stage-Generative-Art) | Suprematist composition |
| [`Layered-Diagram-Generative-Art`](https://github.com/reyrove/Layered-Diagram-Generative-Art) | Layered linework composition |
| [`Japanese-Patterns-Generative-Art`](https://github.com/reyrove/Japanese-Patterns-Generative-Art) | Traditional motif composition |
| [`Matrix-Rain-Generative-Art`](https://github.com/reyrove/Matrix-Rain-Generative-Art) | Digital rain composition |
| **`Organic-Shapes-Generative-Art`** | Layered petal-form composition *(this repo)* |

All fourteen share the same catalogue template, design tokens, and seed-based reproducibility model.

---

## Credits

- **Design & Generative System** — Reyhaneh Daneshdoost
- **Typefaces** — Cormorant Garamond, DM Mono
- **Engine** — p5.js 1.9.0
- **Palettes** — Ice · Fire · Strawberry · Forest · Sunset
- **Edition** — Organic Shapes, Autumn 2026

---

## License

All artwork, code, and generative systems are the intellectual property of the artist. Compositions may not be reproduced, redistributed, resold, or adapted without a written license.

For licensing, commissions, or private systems: **reyhanehdaneshdoost@gmail.com**
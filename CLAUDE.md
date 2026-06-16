# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static HTML site for interactive, animated demonstrations of middle school math models. No build tools, no dependencies, no server — open HTML files directly in a browser.

## How to view

Open any `.html` file in a browser. No dev server needed.
On Windows: `start index.html` or `start 01-functions.html`

## File structure

```
index.html          — Landing page with grid of card links to sub-pages
01-functions.html   — Linear, quadratic, inverse proportion (coordinate-system canvas)
02-triangles.html   — Handshake, three-equal-angles, A/X similarity, midline extension
03-circles.html     — Vertical diameter, inscribed angle, tangent, power of a point
04-optimization.html— General's horse (将军饮马), Hubugui (胡不归), Apollonius, Fermat point
05-special.html     — Half-angle, GuaDou principle (瓜豆), rotation congruence
```

Every file is self-contained: HTML + inline `<style>` + inline `<script>`. No shared JS or CSS files.

## Architecture patterns (shared across all sub-pages)

### Canvas rendering

Two coordinate strategies are used:

- **`01-functions.html`**: Math Cartesian coordinates via `OX, OY, SCALE`. All drawing calls use `tx(x)`/`ty(y)` to map math coords → canvas pixels. Grid lines, axes, and labels are rendered each frame.
- **`02–05`**: Fixed design coordinate system `DESIGN_W=750, DESIGN_H=600`. Canvas is Retina-scaled (`canvas.width = cw * dpr`), then a uniform transform `ctx.setTransform(sX, 0, 0, sY, 0, 0)` maps design coords to the physical canvas. All drawing uses design coords directly.

### UI pattern

Each sub-page has three rendering zones that swap roles on mobile vs desktop:

| Zone | Desktop (≥769px) | Mobile (≤768px) |
|---|---|---|
| Sidebar | Fixed 290px left column | Hidden |
| Canvas | Flex-1 fill remaining space | Flex-1 fills screen |
| Mobile tabs | Hidden | Horizontal scroll tab bar |
| Mobile panel | Hidden | Collapsible bottom sheet for controls |

The `buildUI()` function generates all sidebar and mobile-panel DOM via `innerHTML`. Event handlers use `querySelectorAll` with classes (not IDs) so both sidebar and mobile-panel controls work simultaneously. This was a deliberate fix — never use getElementById for dynamically duplicated UI.

### Animation loop

Standard `requestAnimationFrame` loop: `animate(ts)` converts timestamp to seconds, calls `render()`, which clear→draws based on `currentModel` dispatch.

### Shared drawing primitives

Most sub-pages define these helper functions (minor per-page variation):
- `dp(x, y, r, color, label, glow)` — draw a filled+stroked circle dot, optionally with glow and text label
- `dl(x1, y1, x2, y2, color, width, dashed)` — draw a line segment, optionally dashed
- `glowPt(x, y, r, color)` — concentric fading circles behind a point
- `addTrail(x, y)` / `drawTrails()` — fading particle trail system (golden yellow), max 40-60 particles

### Parameter/speed controls

Speed slider and per-model parameter sliders (`k`, `b`, `a`, `B`, `c`, `K`) sync across both sidebar and mobile panel by querying all matching `.speed-slider` / `.param-slider` / `.speed-val` / `.param-val` selectors and setting values on all.

## Color conventions

- `#60a5fa` (blue) — primary geometry elements (triangles, lines, points)
- `#f472b6` (pink) — transformed/rotated counterparts, secondary emphasis
- `#fbbf24` (golden) — dynamic moving point P, trails, key relationships
- `#34d399` (green) — verification indicators (equality confirmation), right angles
- `#f8fafc` (white) — fixed reference points (O, vertices)
- `#a78bfa` (purple) — vertex, symmetry axis
- `#94a3b8` (gray) — descriptive text, less important info

## Key recent changes

- **Canvas Retina rendering**: Canvas physical pixels set to `clientWidth * devicePixelRatio`, then CSS size constrained. This makes lines sharp on high-DPI screens.
- **class-based selectors**: Moved from `getElementById` to `querySelectorAll('.class')` because `buildUI()` writes the same controls into sidebar AND mobile panel — duplicate IDs broke event binding.
- **Responsive layout**: `clamp()` for fluid typography/spacing, mobile bottom-sheet pattern at ≤768px breakpoint.

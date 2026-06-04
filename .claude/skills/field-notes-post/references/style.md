# Style DNA — 原理手记 / Field Notes

The look that makes every post read as one blog. Internalize this; you should **not** need to
open old post files to match the style — start from `assets/post-skeleton.html`, which already
encodes all of it.

## Palette (warm dark, terracotta accent)

Dark by default, light theme via `[data-theme="light"]`. **Only use the CSS variables** — never a
raw hex.

- `--bg` near-black warm `#0d0e10` / light parchment `#f4f0e7`
- `--ink` / `--ink-dim` / `--ink-faint` — text from primary → muted
- `--accent` terracotta ember `#c8643c` — the one hero color; emphasis, active states, highlights
- `--accent-soft` `#e0a07f` — `<b>` emphasis, secondary accent
- `--sage` `#7f9183` — cool counterpoint (a *second* accent for "the other thing" in a contrast)
- `--line` / `--line-strong` — hairline borders
- Atmosphere is part of the brand: fixed radial gradients + grain noise + faint horizontal grid.

## Type

- `--serif` **Fraunces + Noto Serif SC** — headings, hero, big numbers, the *display* voice.
  Headings are weight 300, slightly negative letter-spacing; the hero accent word is `<em>` italic.
- `--sans` **Noto Sans SC** — body prose (`.desc`), weight 300–400, line-height ~1.9.
- `--mono` **IBM Plex Mono** — labels, captions, metadata, tags, code, numbers-in-grids. Uppercase
  + wide letter-spacing for kickers/captions.

## Voice

- **Chinese prose; English terms kept** (Transformer, softmax, KV cache), glossed in Chinese on
  first mention. Headings often pair a Chinese phrase with an English name in the `.stage-en` slot.
- Confident, concrete, a little warm. Lead with the mechanism + a worked example. `<b>` marks only
  the load-bearing phrase in a sentence — don't over-bold. Wrap real terms/formulas in `<code>`.

## Motion

- One orchestrated entrance: `@keyframes rise` (fade + translateY) staggered via `.d1/.d2/.d3`.
- Scroll-driven reveal: `.reveal` is hidden until its stage gets `.in`. Per-stage payoff animations
  fire once from `runStageEffect(i)`.
- Always leave a static fallback — the stylesheet disables motion under `prefers-reduced-motion`,
  and JS-driven final states must apply even when transitions are off.

## Stage skeleton (fixed rhythm)

```
.stage-head  →  [stage-no] [stage-en] [stage-phase badge]
h2.reveal.d1
p.desc.reveal.d1     (one or two)
.viz.reveal.d2       (one or more figures: .viz-cap + a component + optional .note)
```

## Component vocabulary (all in the skeleton — reuse before inventing)

| class | use |
|---|---|
| `.chips` / `.chip` | small mono tags / token pills |
| `.probs` / `.prob` | horizontal bars for a distribution; animate with `animateProbs(scope)` |
| `.facts` / `.fact` | grid of big serif numbers + mono caption (scale/stats) |
| `.learns` / `.learn` | tags with a leading `<i>` accent label (roles, capabilities) |
| `.steps` / `.step` | numbered vertical list with a connecting line (a loop / pipeline) |
| `.matblock` + `makeMatrix()` | render matrices (rows of `.mcell`); `.hl`/`.hl2`/`.masked`/`.lo` cell states |
| `.attn` `.c` | heatmap grid; set cell `background` from data on reveal, add `.lit` for legible text |
| `.note` | a mono footnote/aside under a figure |
| `.viz` / `.viz-cap` | the framed figure container + its uppercase caption |

If a topic needs a bespoke visual, build it from the same tokens (terracotta fills, hairline
borders, mono labels, SVG/CSS not images, animate-on-reveal) and add its CSS in the post's own
`<style>`. Good examples to imitate: a descending SVG loss curve, an attention arc diagram, a
weighted-sum "mixing" stack — all are just tokens + a draw/`fade` animation.

## Two layouts

- **Scrollytelling** (the skeleton): sequential process, one `.stage` per step, scroll-driven.
  Mind the index coupling: stage DOM order, `railLabels`, the topbar `/ NN` total, and the
  `runStageEffect` `if(i===N)` blocks are all 0-based-position-linked and must stay in sync.
- **Editorial**: shorter/non-sequential. Keep `<head>` + topbar + atmosphere from the skeleton,
  drop rail/spine/progress/autoplay + their script, and lay out plain `<section>`s of
  `h2` + `.desc` + `.viz`. Same components, same tokens.

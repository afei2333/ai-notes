# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal **technical blog** — a slowly growing collection of in-depth, interactive explainers
(in Chinese) about how complex systems actually work. The first article walks through the full
lifecycle of a large language model; future articles will cover other topics (attention,
backprop, tokenizers, …), added whenever there's something worth explaining well.

Two layers:

- **`index.html`** — the archive cover / table of contents ("Field Notes" / 原理手记). It renders
  the article list **from a `POSTS` manifest** in its inline script (not hand-written `<li>`s) and
  provides **client-side search + tag filtering** over that manifest.
- **`posts/<slug>.html`** — one self-contained file per article (e.g. `posts/llm-lifecycle.html`),
  all living under the `posts/` directory. Each post is its own HTML + inline CSS + vanilla JS, no
  framework, no shared asset files.

The repo layout:

```
index.html            # archive cover / table of contents (entry point, stays at root)
posts/                # all article HTML lives here
  llm-lifecycle.html
  attention-mechanism.html
.claude/              # project skill + skeleton (field-notes-post)
CLAUDE.md  README.md  LICENSE
```

The guiding ethos for every post is **pedagogical depth**: a reader should come away genuinely
understanding the mechanism, not just having toured it. Favor concrete mechanism over
hand-waving — show the loop, name the real terms (cross-entropy, backprop, AdamW, KV cache),
and give the reader a worked example they can hold onto.

## Running

No build step, no dependencies to install. Open a file directly:

```bash
open index.html                  # the blog home
python3 -m http.server 8000      # then visit http://localhost:8000/
```

External dependencies are kept to CDN `<link>`/`<script>` tags only — no build step, no package
manager. Every page loads **Google Fonts**; **math-heavy posts may also load KaTeX** (see "Rendering
math" below). Everything else — styles, animation logic, SVG visuals — is inline. No package manager,
linter, or test suite exists.

## The house style (shared design system)

Visual consistency across posts is what makes this read as one blog rather than scattered pages.
Every page repeats the same foundation, copied inline (there is no shared stylesheet):

- **Design tokens** — a `:root` block of CSS custom properties (dark theme) overridden under
  `[data-theme="light"]`. **Never hardcode colors**; always use the variables: `--accent`
  (terracotta ember), `--sage`, `--ink` / `--ink-dim` / `--ink-faint`, `--line` / `--line-strong`,
  `--bg*`, and the `--serif` / `--sans` / `--mono` font vars.
- **Typography** — Fraunces + Noto Serif SC for headings/display (serif), Noto Sans SC for body,
  IBM Plex Mono for labels, captions, and metadata. Loaded from the same Google Fonts URL.
- **Rendering math** — do **not** hand-build formulas out of `<sub>`/`<sup>`/Unicode (`ᵀ`, `√`, `ₖ`)
  or wrap them in mono `<code>`; that mixing is what makes a page's typography look chaotic. Posts
  with real math load **KaTeX** (CDN css + `katex.min.js` + `auto-render.min.js`, all `defer`) and
  write math as `\( … \)` (inline) / `\[ … \]` (display). `attention-mechanism.html` is the
  reference: it calls `renderMathInElement(main, {delimiters, trust:true, strict:false})` from a
  `DOMContentLoaded` handler (the `defer` scripts are ready by then), and renders the hero formula
  explicitly with `katex.render(…, {displayMode:true, trust:true})`, color-coding parts via
  `\htmlClass{m-q|m-k|m-v}{…}` mapped to `--accent` / `--sage` / `--accent-soft`. Keep prose Chinese
  and tokens (猫/很/累) outside math — KaTeX has no CJK glyphs, so CJK in `\text{}`/subscripts won't
  render; show token-subscripted expressions as their own HTML/SVG component instead.
- **Atmosphere** — the fixed `.atmosphere` radial gradients, `.grain` noise overlay, and
  `.grid-lines` are part of the brand; carry them into new posts.
- **Theme toggle** — a topbar button that persists choice in `localStorage` under the key
  **`llm-theme`**. This key is shared across `index.html` and every post, so dark/light follows
  the reader between pages. Reuse the same key; don't invent a new one.
- **Back-to-home link** — every post's topbar **must** include a `.home-link`
  (`<a class="home-link" href="index.html">← 原理手记</a>`, wrapped with the wordmark in a
  `.top-left` container) so readers can always return to the archive. This is part of the shared
  topbar chrome; carry it into every new post.

When starting a new post, copy the chrome from the canonical skeleton
**`.claude/skills/field-notes-post/assets/post-skeleton.html`** (a ready-to-fill template carrying
the full design system, topbar, scroll engine, and reusable component library) rather than writing
it fresh or re-deriving it from an existing post. The style is summarized in
`.claude/skills/field-notes-post/references/style.md`. Keep that skeleton in sync if the house
style ever changes — it is the source of truth for post chrome.

## The collaboration workflow (how posts get written)

> This loop is also packaged as the **`field-notes-post`** project skill
> (`.claude/skills/field-notes-post/`), which triggers when the user hands over a topic/angle to
> turn into an article. The skill is the procedure; this file remains the source of truth for
> conventions.

The intended loop, when the user brings a new angle/topic ("切入点"):

1. **Draft the content first, in chat.** Turn the angle into an article outline + the actual
   prose/figures to the depth bar above, and confirm it with the user **before** writing files.
   Don't generate the HTML until the content is agreed.
2. **Generate `posts/<slug>.html`** following the conventions below.
3. **Register the post** by adding it to the `POSTS` manifest in `index.html` (next step).
4. **Verify** in a browser (see the sanity-check list).

## Authoring a new post

1. **Create `posts/<slug>.html`** under the `posts/` directory with a short, kebab-case, English
   slug (`attention-mechanism.html`, `backprop.html`). Start from an existing post's skeleton so
   the house style carries over. Because posts live one level below the root, the topbar
   back-link must point to **`../index.html`** (not `index.html`).
2. **Write the article** to the depth bar above. Pick the layout that fits the topic — see
   "Article layouts" below.
3. **Register it in the `POSTS` manifest** inside `index.html`'s `<script>` — this is the single
   source of truth for the archive; the entry cards, the search index, the tag-filter row, and
   the masthead count are all derived from it. Add one object:
   ```js
   { no:'05', status:'published', slug:'posts/<slug>.html', featured:false,
     kicker:'…', title:'…', dek:'一句话钩子',
     tags:['标签A','标签B'], meta:['版式','约 N 分钟'] }
   ```
   - `status:'published'` renders a clickable card and needs `slug`; `status:'soon'` renders a
     dimmed `即将刊出` teaser (only `no/title/dek/tags`).
   - `tags` automatically join the global tag-filter row and the search index — no extra wiring.
   - The masthead "篇已刊出" count and the `N 篇` contents count update themselves from the
     manifest; **do not hardcode counts**.
   - Do **not** hand-write entry HTML in `index.html` anymore — the old static `<a class="entry">`
     blocks were replaced by manifest rendering.
4. **Sanity-check** in a browser: search + tag filtering find the new post, theme toggle persists
   across the index↔post jump, reveal animations fire, and `prefers-reduced-motion` still shows
   all content.

## The archive (`index.html`) — how search & tags work

All data-driven from the `POSTS` array; the inline script:

- Renders each post into `#entries` via `entryMarkup(p)` (string template, run through `esc()` —
  keep using it since manifest strings may contain `<`, `&`, quotes). Each card stores a
  lowercased `data-search` blob (title + dek + tags) and a `data-tags` (`|`-joined) for filtering.
- Builds the tag-filter row from the unique tags across all posts (first-seen order).
- `applyFilter()` is the one filter path: a post shows if it matches the search text **and**
  (no tags selected **or** it has at least one selected tag). Search and tags compose. It also
  updates the count label and toggles the empty state.
- Tag pills are clickable both in the toolbar **and** inside entry cards (a delegated document
  click handler; clicks on a tag inside an `<a class="entry">` call `preventDefault()` so they
  filter instead of navigating). `Esc` / the ✕ button clears the search.

Filtering only toggles `display` on already-rendered cards — the staggered entrance animation
plays once on load and isn't re-triggered. When changing entry markup, update `entryMarkup`, not
static HTML.

## Article layouts

A post does not have to be a scrollytelling journey. Choose by topic:

- **Scrollytelling stages** (used by `llm-lifecycle.html`) — best when the subject is a
  sequential process with distinct steps. See the next section for how its engine works and its
  one sharp gotcha.
- **Plain editorial article** — for shorter or non-sequential explainers, a straightforward
  flow of headings + `.desc` prose + `.viz` figures is perfectly fine. Reuse the same component
  classes so it still looks like the blog.

Whichever you choose, reuse existing component classes (`.viz`, `.chips`, `.probs`, `.steps`,
`.facts`, `.learn`, `.promptbox`, `.pairs`, `.ranked`, …) before inventing one-off styles.

## The scrollytelling engine (reference: `llm-lifecycle.html`)

The lifecycle post renders as 11 `<article class="stage">` sections inside `<main class="shell">`,
driven by one IIFE in `<script>`.

Reveal is **scroll-driven**. All stages live in the flow from load; an `IntersectionObserver`
adds `.in` to each stage as it scrolls into view, triggering its CSS reveal animation and firing
that stage's one-time effect via `runStageEffect(i)`. A scroll listener tracks the "current"
stage (top above ~45% of the viewport) to drive the progress counter, the lit spine, and the
side-rail active state; a second observer reveals the finale at the bottom. There are **no
manual advance buttons** — the hero "开始旅程" button smooth-scrolls to stage 0, the autoplay
button (`autoTick`) hands-free auto-scrolls stage by stage, and replay clears `.in` and returns
to top.

### Index coupling — read before adding/removing/reordering a stage

Within a scrollytelling post the stages are tied together by **positional index** in several
places that must stay in sync. Changing the count or order means updating all of these:

- The DOM order of `<article class="stage">` elements.
- The `railLabels` array (side-nav labels) — must match stage count and order.
- The hardcoded total in the topbar (`00 / 11`).
- `runStageEffect(i)` — the set of `if(i===N)` blocks that fire each stage's animation
  (e.g. `i===4` drives the pre-training prob bars + masked-blank reveal, `i===8` the attention
  heatmap, `i===10` the streaming output). These indices are **0-based** and refer to position,
  not the displayed stage number.

### Per-stage visuals — the build/animate split

Two kinds, and new visuals should follow the same split:

- **Built once** at load via small IIFEs at the bottom of the script (funnel, transformer layer
  grid, server nodes, attention map). They populate empty container elements by `id`.
- **Animated on reveal** in `runStageEffect(i)` (prob-bar fills, typewriter prompt, attention
  alpha sweep, token streaming). Helpers: `animateProbs`, `typeText`, `animateAttention`,
  `streamOutput`.

Build the static structure once; trigger motion from `runStageEffect` keyed to the stage index.

### Effects must replay on every scroll-in — not once

**Animations are not one-shot.** A reader who scrolls a visual out of view and back must see it
**re-run from the start**, every time. Do **not** gate effects behind a "has it fired before?"
flag (the old `!el.classList.contains('in')` one-time pattern). Instead:

- The `IntersectionObserver` **adds** `.in` + resets + fires the effect when a stage enters view,
  and **removes** `.in` + resets when it leaves — so both the CSS reveal and the JS effect re-arm.
- Pair every `runStageEffect(i)` with a `resetStageEffect(i)` that restores the stage's
  pre-animation state (clear inline styles, remove highlight classes, reset counters to their
  static final value).
- Make effects cancellable: track each stage's `setTimeout` / `requestAnimationFrame` handles and
  clear them on reset, so a fast scroll never leaves a half-played animation or a stuck-invisible
  cell. (`attention-mechanism.html` is the reference implementation: `timers`/`rafs` maps,
  `clearStage`, `later`, `countUp`.)
- The reset's restored state **is** the `prefers-reduced-motion` / no-JS fallback — it must show
  the complete, correct final content on its own.

## Prose & content conventions

- **Prose is Chinese**; technical terms keep their English (token, Transformer, RLHF), usually
  paired with a Chinese gloss on first use. Headings use the serif font; captions/labels use mono.
- Use `<b>` for emphasis on the load-bearing phrase in a sentence — don't over-bold.
- Lead with the mechanism and a concrete, memorable example. When you state a number or a claim,
  make it specific (real orders of magnitude, real term names).
- Respect `prefers-reduced-motion` — the stylesheet already disables animation under it; never
  add motion that only works through JS transitions with no static fallback.

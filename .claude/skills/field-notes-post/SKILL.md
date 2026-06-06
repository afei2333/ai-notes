---
name: field-notes-post
description: >-
  Write and publish a new article for THIS repo's Chinese technical blog ("原理手记 / Field Notes" —
  index.html + per-post <slug>.html files). Use this whenever the user hands over a topic, angle, or
  "切入点" to turn into a post, or asks to write / add / draft / publish a new article or explainer for
  the blog — e.g. "写一篇讲 KV 缓存的博客", "把这个做成一篇文章", "新增一篇讲反向传播的", "add a post
  about tokenizers", "give me an article on RoPE". Also use when registering a finished post into
  index.html's POSTS manifest, or when updating the archive list, tags, or search entries. Trigger
  even if the user just describes a concept and says "make this into a post" without naming files.
  The skill enforces the house style, the content-confirm-first workflow, and manifest registration.
---

# Field Notes — author a new blog post

This skill turns a topic into a published article for the blog in this repo. Read `CLAUDE.md` at
the repo root first — it is the source of truth for architecture and conventions; this skill is the
*procedure* that applies them.

The blog has two layers:
- **`index.html`** — the archive cover. Renders the article list from a `POSTS` manifest in its
  inline `<script>`, and provides client-side search + tag filtering.
- **`posts/<slug>.html`** — one self-contained article per file (inline CSS + vanilla JS, no
  framework), living under the `posts/` directory. Its topbar back-link points to `../index.html`.
  The canonical starting point is **`assets/post-skeleton.html`** (bundled with this skill); it
  carries the house style so you don't need to read existing posts. `llm-lifecycle.html` and
  `attention-mechanism.html` are worked examples to imitate if you want a reference.

The whole point of the blog is **pedagogical depth**: a reader should finish genuinely understanding
the mechanism, not just having toured it. Every decision below serves that.

## The workflow

### Phase 1 — Shape the content, and confirm it BEFORE writing files

This is the most important rule: **do not generate any HTML until the content is agreed.** The user
wants to steer the substance first. So:

1. Take the angle and turn it into an **outline + the actual prose and figures** — in chat, as
   markdown. Include: the hook, the 4–8 sections (or scrollytelling stages), and for each the real
   explanation — the mechanism, the named terms, and at least one concrete worked example the reader
   can hold onto. Where a visual would teach better than words, describe what it shows.
2. Propose the post's **metadata**: a short kebab-case English `slug`, a one-sentence dek (hook),
   3–6 tags, and a reading-time estimate.
3. Present it and ask the user to confirm or adjust. Iterate on the *content* here — it's cheap.
   Only move to Phase 2 once they're happy.

Hold the depth bar while drafting: favor concrete mechanism over hand-waving, name the real terms
(cross-entropy, backprop, AdamW, KV cache, …), give worked examples, and when you cite a number make
it specific and real. Vague tours are the failure mode to avoid.

### Phase 2 — Build `posts/<slug>.html`

1. **Start from the bundled skeleton — don't go read old post files.** Copy
   `assets/post-skeleton.html` (relative to this skill) to `posts/<slug>.html`. It already encodes
   the
   whole house style: `<head>` + Google Fonts, the full `:root`/`[data-theme="light"]` token
   system, atmosphere/grain/grid, the topbar (**theme toggle + its script + the back-to-home
   `.home-link`** — the localStorage key is **`llm-theme`**, shared across all pages; never rename
   it), the side rail, the progress bar, the **scroll-driven reveal engine**, the reusable
   component library, and the finale. Then fill in the `TODO` markers. Read `references/style.md`
   once for palette/voice/component vocabulary — that plus the skeleton is all you need to match
   the look; you should not have to open another post.
2. **Honor the tokens.** Never hardcode colors; always use the CSS variables (`--accent` terracotta,
   `--sage`, `--ink*`, `--line*`, `--bg*`, and the `--serif`/`--sans`/`--mono` font vars). Headings
   use the serif font; captions/labels use mono. (The skeleton already does this — keep it that way.)
3. **Pick the layout** that fits the topic (see "Choosing a layout"). The skeleton is the
   scrollytelling variant; for an editorial post, strip the rail/spine/progress/autoplay as the
   note at the top of the skeleton explains.
4. **Write the body** to the depth bar, with `<b>` reserved for the load-bearing phrase in a
   sentence. Prose is Chinese; technical terms keep their English, glossed in Chinese on first use.
   Reuse the skeleton's components (`.probs`, `.facts`, `.steps`, `.matblock`+`makeMatrix`, `.attn`,
   `.chips`, `.learns`) before inventing one-off styles.
5. **Keep the stage count consistent** — `railLabels`, the topbar `/ NN` total, the stage DOM
   order, and the `runStageEffect(i)` index blocks must all agree (0-based position).
6. **Respect `prefers-reduced-motion`** — the skeleton already disables animation under it; never
   add motion that only works through JS transitions with no static fallback.

### Phase 3 — Register the post in the `POSTS` manifest

`index.html`'s `POSTS` array is the single source of truth for the archive — the entry cards, the
search index, the tag-filter row, and the masthead count all derive from it. **Do not hand-write
entry HTML; add one object.**

```js
{ no:'05', status:'published', slug:'posts/<slug>.html', featured:false,
  kicker:'交互长文 · Interactive',        // small label above the title
  title:'…', dek:'一句话钩子',
  tags:['标签A','标签B'],                  // auto-join the filter row + search
  meta:['版式','约 N 分钟'] }              // shown in the entry footer
```

- Use `status:'published'` with a `slug` for a live post; `status:'soon'` (only `no/title/dek/tags`)
  for a forthcoming teaser.
- If a matching `status:'soon'` placeholder already exists for this topic, convert it in place
  rather than adding a duplicate.
- Counts update themselves — the masthead "篇已刊出" and the contents "N 篇" read from the manifest.
  **Never hardcode a count.**
- New tags need no extra wiring; the filter row and search index rebuild from `POSTS` on load.

### Phase 4 — Verify in a browser

Open the site (e.g. `python3 -m http.server 8000`) and check:
- The new post appears in the archive; **search and the tag filter find it**.
- Its tags toggle the filter; the count label and empty-state behave.
- The **theme toggle persists across the index↔post jump** (shared `llm-theme` key).
- Reveal animations fire, and with `prefers-reduced-motion` on, all content is still visible.

## Choosing a layout

A post does **not** have to be a scrollytelling journey. Match the form to the topic:

- **Scrollytelling stages** (like `llm-lifecycle.html`) — best when the subject is a *sequential
  process* with distinct steps. Reveal is scroll-driven via the skeleton's enter/leave
  `IntersectionObserver`, which **replays** each stage's effect on every scroll-in: it fires
  `runStageEffect(i)` on enter and `resetStageEffect(i)` on leave (the engine — `timers`/`rafs` +
  `clearStage` — is already in the skeleton; you just fill the per-index branches). ⚠️
  **Index-coupling gotcha:** the stages are tied together by 0-based position in several places that
  must stay in sync — the DOM order of `<article class="stage">`, the `railLabels` array, the
  hardcoded `00 / NN` total in the topbar, and the `if(i===N)` blocks in **both** `runStageEffect`
  **and** `resetStageEffect`. Changing the count or order means updating all of them. Build static
  structure once (small IIFEs at the bottom of the script) and trigger motion from `runStageEffect`
  keyed to the index.
- **Plain editorial article** — for shorter or non-sequential explainers, a straightforward flow of
  serif headings + `.desc` prose + `.viz` figures is perfect, and avoids the index-coupling burden.
  Reuse the same component classes so it still reads as the blog.

When in doubt, prefer the editorial layout — it's simpler and lower-risk. Reach for scrollytelling
only when the journey itself is the lesson.

## Tips that keep posts feeling like one blog

- Reuse existing component classes before inventing one-off styles; if you genuinely need a new
  visual, build it from the same tokens and add its CSS in the post's own `<style>`.
- SVG and CSS visuals beat images — they theme correctly and stay crisp. Animate on reveal, with a
  sensible static fallback.
- Keep the editorial voice: confident, concrete, a little warm. The reader came to *understand*.

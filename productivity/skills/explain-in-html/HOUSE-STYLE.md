# House style

Every page in this system reads as though the same person made it. The exemplars carry the CSS; this file carries the rules the CSS can't state — what the tokens mean, when each component applies, and what must never appear.

## Tokens

Paste this block into `:root` unchanged. Never introduce a hex value outside it; a color the palette doesn't have is a signal you're reaching for a component the house style doesn't have.

```css
:root {
  --ivory:   #FAF9F5;   /* page background */
  --slate:   #141413;   /* headings, emphasis, dark panels */
  --clay:    #D97757;   /* the one accent — see below */
  --oat:     #E3DACC;   /* soft fills: callouts, carryover, avatars */
  --olive:   #788C5D;   /* good: pass, resolved, healthy, improved */
  --rust:    #B04A3F;   /* bad: fail, error, regression */
  --gray-150:#F0EEE6;   /* subtle fills, table headers, gridlines */
  --gray-300:#D1CFC5;   /* borders, rules */
  --gray-500:#87867F;   /* labels, captions, secondary text */
  --gray-700:#3D3D3A;   /* body text */
  --serif: ui-serif, Georgia, "Times New Roman", serif;
  --sans:  system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  --mono:  ui-monospace, "SF Mono", Menlo, Consolas, monospace;
}
```

The report exemplars call `#F0EEE6` `--gray-100`. It is the same color; rename it to `--gray-150` when you adapt them so every page names it once.

Two extensions, and only these two:

- **Series colors**, when data has more categories than the palette has meanings — a ring's owners, a multi-series chart. Cycle in order from clay: `#D97757 #788C5D #6A8CAF #C2A83E #B04A3F #87867F #3D6E6E #A67C52`.
- **Diff tints**, legible on a `--slate` code panel: `#E0897A` for removed lines, `#A3B88A` for added.

**Clay is a spotlight, not a brand color.** It marks the single thing that matters most in each region: the active tab, the peak bar in a chart, the current node, the accent edge of the TL;DR. If two things on one screen are clay, neither reads as important. Olive and rust carry judgment — never use clay where the meaning is "good" or "bad".

## Type

Three faces, three jobs, no overlap:

- **serif** — `h1`, `h2`, big stat numbers, `<summary>`, glossary and side-panel titles. Weight 500, `letter-spacing: -0.01em`.
- **sans** — body prose at 15px/1.6, table cells, list items.
- **mono** — eyebrows, section labels, pills, timestamps, `file:line` stamps, code. Labels get `font-size: 10–11px`, `letter-spacing: 0.06–0.1em`, `text-transform: uppercase`, `color: var(--gray-500)`.

Prose is capped at `max-width: 680px` regardless of how wide the page runs.

## Layout

- `body { background: var(--ivory); padding: 56px 24px 120px }`.
- One centered container: 820–860px for reports, 980–1100px when a sticky nav, glossary, or detail panel sits beside the content.
- Panels are `background: #fff; border: 1.5px solid var(--gray-300); border-radius: 12–14px`. Reserve `--slate` panels for the two places darkness carries meaning: an incident TL;DR and a code/diff block.
- Sticky sidebars use `position: sticky; top: 24–32px; align-self: start`.
- One breakpoint at 760–960px collapsing the grid to a single column. Sidebars either hide (nav) or reflow below (glossary, detail panel).

## Components

Reach for these before inventing anything. Each one lives in at least one exemplar — copy it from there rather than writing it fresh.

| Component | Where | Use for |
|---|---|---|
| eyebrow | all explainers, flowchart | mono kicker naming the page's category |
| TL;DR | explainers (bordered), incident (dark) | the whole answer in 2–4 sentences |
| `<details>` step | feature-explainer | a stage of a process, with `file:line` right-aligned in the summary |
| code tabs | feature-explainer | the same thing shown from 2–4 angles: config, call site, response |
| callout | feature-explainer | one aside per page, maximum |
| glossary aside | concept-explainer | 4–6 terms, hover-linked from `.term` spans in prose |
| stat band | status-report | exactly four numbers, each with a delta line |
| risk dots | status-report | olive/clay/rust severity in a table cell |
| pills | incident-report | severity, state, and 2–3 neutral facts in the header |
| timeline | incident-report | ordered events; clay dot at impact, olive at mitigated |
| diff panel | incident-report | the actual change that caused it, ±3 lines of context |
| action items | incident-report | owner initials, description, due date, done state |
| detail panel | flowchart | per-node depth without leaving the diagram |

## SVG

Diagrams are hand-authored inline SVG. No chart library, no mermaid, no image files.

- Author against a `viewBox` and let CSS size it: `width: 100%; height: auto`.
- Flat fills, 1.5–2px strokes, palette colors only.
- Define `<marker>` arrowheads in `<defs>` — one per edge color.
- Label directly on the drawing. A diagram that needs a legend to be understood usually needs a redraw; a legend that names *categories* is fine.
- Interactive nodes: `<g class="node" data-k="…">`, a `DETAIL` map in JS, and a click handler that swaps the side panel. The exemplar's ~20 lines of JS is the whole pattern.
- Charts are hand-computed geometry — plot the numbers, label every bar, and use `--oat` for the series with `--clay` for the one bar worth noticing.

## Hard constraints

- **One file.** All CSS in one `<style>`, all JS in one `<script>`, no build step.
- **No network.** No CDN scripts, no external stylesheets, no font imports, no remote images. The page must render identically with the machine offline.
- **Vanilla JS.** No framework, no bundler. If a feature needs more than ~40 lines, it's probably depth that belongs in a `<details>` instead.
- **Semantic HTML.** `<header> <nav> <main> <section> <figure> <dl> <table> <footer>`. Headings descend without skipping.
- **`<title>`** names the topic, not the doc type: "How rate limiting works in acme/api", not "Feature explainer".

## Self-check

Run all of these before handing the page over. Each is checkable — do not report the page finished with any of them unresolved.

1. `grep -nE 'https?://|@import|cdn\.' <file>` returns nothing but genuine content links in prose or the footer.
2. No `<!-- exemplar · … -->` marker survives from the template.
3. Every hex in the file appears in the token block, the series set, or the diff tints — chart and SVG fills included. White is the one addition, for panel backgrounds; write it `#FFFFFF` and normalize the exemplars' `#fff` shorthand when you adapt them, so one audit catches every color literal:

   ```bash
   grep -ohE '#[0-9A-Fa-f]{6}\b|#[0-9A-Fa-f]{3}\b' <file> | sort -u
   ```

   The audit reads text as well as CSS, so a page that *writes about* colors — a palette table, a diff quoting a stylesheet — reports hits it should keep. Clear each hit as either an allowed value or visible content; only style values have to be in the set.
4. Every section of the spine for this doc type is present and filled from real sources — no lorem, no "TBD", no invented names or numbers.
5. Every `file:line`, PR number, metric, and timestamp on the page came from something you read.
6. The `<footer>` names the sources.
7. At the 720px width the layout collapses to one column with no horizontal scroll; wide tables and code blocks scroll inside their own container.
8. The page opens in a browser and its interactive parts — tabs, `<details>`, node clicks, sliders — respond.

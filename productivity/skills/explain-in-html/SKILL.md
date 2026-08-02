---
name: explain-in-html
description: Turn a topic into one self-contained HTML page — feature or concept explainer, design doc, status or incident report, flowchart, or SVG illustration set.
disable-model-invocation: true
---

# Explain in HTML

Turn a topic — a feature, a concept, a system's design, a week of work, an incident, a process — into **one** self-contained HTML page, cut to the **house style** and optimized to be **read once**.

## Read-once

The reader opens this page a single time and never comes back. That decides every content call:

- **Answer first.** A TL;DR at the top states the conclusion before any evidence for it.
- **Every section stands alone.** Nothing on the page requires having read a section further down.
- **Show the thing.** A diagram, an annotated snippet, or a table earns its space where a paragraph describing the same thing does not.
- **Depth goes behind a click.** `<details>`, tabs, and detail panels keep the one-pass path short while the curious reader can still go down.

## Grounded

Every fact on the page traces to something you actually read — a file, a diff, a log, a doc, a URL the user gave you. Two visible consequences, both required:

- Step and section headers carry their provenance inline: `middleware/ratelimit.ts:21`, `PR #4888`, `cfg-9a12`.
- The page ends with a `<footer>` naming its sources, and explainers list the files read in the sticky nav.

When you can't source a claim, drop the claim. When a section can't be filled from sources, say what's unknown in the page rather than inventing filler.

## Steps

### 1. Settle the brief

You are ready when you can state, in one sentence each: the **doc type**, who reads it, which sources you will read, and what the page must cover.

Bundle everything you'd otherwise guess at into one round of `AskUserQuestion` — doc type when the request is ambiguous between two, audience when it swings the depth, sources when you can't tell what to read, scope when the topic is broader than one page. If all four are already clear from the request, skip the round and build.

### 2. Do the legwork

Read the sources. This is the step that decides whether the page is worth reading, and it is finished only when **every section of your outline has real material behind it** — named files, real numbers, actual quotes, concrete line references. An outline where a section is still a title with nothing under it is an unfinished step, not a section to fill in with plausible prose while writing HTML.

### 3. Build from the exemplar

Read `HOUSE-STYLE.md`, then read the exemplar for your doc type from the table below. Build the page by adapting that exemplar's structure to your material — keep its spine, its component vocabulary, and its CSS token block; replace its content wholesale.

Strip the exemplar's `<!-- exemplar · … -->` marker comment. It identifies a template, and your page is not one.

### 4. Self-check, then hand over

Run every check in `HOUSE-STYLE.md` § Self-check. Then write the file and open it:

```powershell
Invoke-Item docs\explainers\<slug>.html
```

Report the path, the doc type, and the sources you drew on.

## Doc types

| Type | Use when | Exemplar | Spine |
|---|---|---|---|
| **feature-explainer** | "How does X work in our codebase?" | `exemplars/feature-explainer.html` | eyebrow → TL;DR → the path step by step, one `<details>` per stage stamped `file:line` → usage/config tabs → gotchas → FAQ. Sticky nav lists sections and files read. |
| **concept-explainer** | A general idea, not tied to this repo | `exemplars/concept-explainer.html` | lead posing the question → the trick in one sentence → interactive demo → comparison table vs the naive approach → where you'll meet it. Sticky glossary aside; terms in prose get `.term` dotted underlines wired to it. |
| **design-explainer** | Our system's architecture and why it's shaped this way | `exemplars/feature-explainer.html` for the chassis, `exemplars/flowchart.html` for the diagram | TL;DR → architecture diagram with clickable nodes → component walkthrough → request/data flow → **decisions & alternatives** (what was rejected, and why) → constraints & failure modes → open questions. The decisions section is what makes it a design doc; without it you've written a feature explainer. |
| **status-report** | A period of work | `exemplars/status-report.html` | header with date range + repo → four-stat band with deltas → highlights → shipped table with risk dots → velocity chart → carryover tagged in-review/blocked/slipped → sources footer. |
| **incident-report** | An outage or degradation | `exemplars/incident-report.html` | id + title + severity pills → dark TL;DR → timeline with impact/mitigated dots → root cause + diff panel → impact table → action items with owner initials and due dates. |
| **flowchart** | A process with branches and failure paths | `exemplars/flowchart.html` | inline SVG: rounded terminals, rect steps, diamond gates, olive `yes` and dashed rust `no` edges → sticky detail panel driven by a `DETAIL` map → legend. |
| **svg-illustrations** | A set of standalone figures for docs | `exemplars/svg-illustrations.html` | one `<figure>` per illustration at a fixed canvas size, caption naming where it will be used, download button per figure. |

Two types in one page is a sign the topic wants two pages. Split it and cross-link.

## Output

Default path: `docs/explainers/<slug>.html`, where `<slug>` is kebab-case from the title. Use a path the user names instead when they name one.

Existing file at that path: read it first, then ask whether to update it in place or write a new one alongside.

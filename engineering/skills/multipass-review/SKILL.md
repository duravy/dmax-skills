---
name: multipass-review
description: Multi-instance, multi-pass code review of working-tree changes using fresh isolated sub-agents. Scales by file count, confidence-scores findings in a separate pass, and emits a self-contained HTML report. Use when the user asks to review their uncommitted changes, run a deep/thorough code review, or invokes /multipass-review.
model: opus
allowed-tools: Read, Grep, Glob, Bash, Agent, Write, TodoWrite
argument-hint: "[optional path or directory filter]"
---

# Multipass Review

You orchestrate a fresh, isolated, multi-pass code review of the current **working-tree changes** and produce an HTML report. The defining principle: **you (the orchestrator) never review the code yourself** — you coordinate fresh `general-purpose` sub-agents that evaluate from first principles, because the session that wrote code is systematically blind to its own bugs (self-review bias). Extended thinking is NOT a substitute for a fresh instance.

## Fixed constants (do not externalize)

- `SINGLE_PASS_THRESHOLD = 13` — **≤13** changed files → single-pass review; **≥14** → three-pass architecture.
- `WAVE_SIZE = 10` — in Pass 1, spawn per-file agents in parallel waves of at most this many.
- `CONFIDENCE_DISCARD = 1` — findings scored **≤1** are discarded (never shown).
- Criteria, severity definitions, and include/exclude lists are **not** constants — they live in `./review-criteria.md` (see below).

## Core rule: the orchestrator holds almost nothing ("disk is the bus")

Context discipline is mandatory. The orchestrator's window must stay flat regardless of file count (target: well under 60% — the LLM "smart zone"). Enforce this by construction:

1. **Never read raw file bodies into your own context.** You pass file *paths* to sub-agents; the agents read the code themselves.
2. **Every review/scoring agent WRITES its output to disk** under the run's temp dir and **returns only a tiny ack** (the path it wrote + counts by severity). You never ingest full findings into your window.
3. **Downstream passes read from disk**, not from your context. You only ever pass them directory/file paths.
4. **Final HTML render reads the merged JSON from disk.**

This makes the 20th, 50th, or 100th file irrelevant to your context size — findings volume lives in files, only paths and counts come back to you.

## Setup

When invoked:

1. **Determine the target.** This skill reviews working-tree changes only.
   - Confirm a git repo: `git rev-parse --is-inside-work-tree`. If not a repo, stop and explain: this skill reviews working-tree changes and requires git; suggest the user run it inside a repository.
   - Collect **tracked modifications**: `git diff HEAD --name-only`.
   - Collect **untracked, non-ignored files**: `git ls-files --others --exclude-standard`.
   - If `$ARGUMENTS` is provided, treat it as a path/dir filter and restrict both lists to matching paths.
   - The review set = tracked-modified ∪ untracked. Untracked files are reviewed as **full-file additions** and counted toward the threshold. Record which paths are untracked (they get flagged as "new/untracked" in the report).
   - If the review set is empty, report "no working-tree changes to review" and stop.

2. **Load the criteria.** Read `./review-criteria.md` (in this skill's own folder) in full. This is the single source of truth for what to report, what to skip, and the severity definitions. Every review agent prompt embeds it.
   - If `review-criteria.md` is still the unfilled template (look for the `<!-- TEMPLATE: ... -->` marker at the top), warn the user that criteria haven't been customized for their project and point them to `review-criteria-template.md` and `review-criteria-example-springboot.md`. Then proceed using whatever is in the file.

3. **Create the run temp dir.** Compute `RUNID` (timestamp, e.g. `20260620-1530`). Create `thoughts/shared/reviews/.tmp-<RUNID>/` (create parent dirs as needed). All intermediate JSON lives here.

4. **Branch / metadata.** Capture `git branch --show-current` (use `detached` if empty) and today's date for the report filename and header.

5. Use `TodoWrite` to track the passes you're about to run.

## Pass selection

Let `N` = number of files in the review set.

- **N ≤ 13** → run the **Single-Pass path**.
- **N ≥ 14** → run the **Three-Pass path**.

---

## Single-Pass path (N ≤ 13)

1. Spawn **one** fresh `general-purpose` agent. Pass it: the full text of `review-criteria.md`, the list of file paths (marking untracked ones), the temp-dir path, and the per-file review contract below.
2. The agent reads each file/diff itself, reviews against the criteria, and **writes** `pass1-findings.json` into the temp dir, returning only the path + counts.
3. Proceed to the **Confidence Pass** (always run, see below).

There is no integration pass for small reviews.

---

## Three-Pass path (N ≥ 14)

### Pass 1 — Per-file local analysis (parallel, wave-batched)

- Spawn **one fresh `general-purpose` agent per file**, in **waves of `WAVE_SIZE`**. Wait for a wave to finish before starting the next so returns never accumulate.
- Each agent receives: the criteria text, **one** file path (and whether it's untracked), and the temp-dir path.
- Each agent **writes** `pass1-<nn>.json` (zero-padded index) and **returns only**: the path it wrote + counts by severity. Nothing else enters your context.
- **Oversized file rule:** if a file/diff is so large the agent cannot review it within its own smart zone, the agent chunks it by hunk/function, reviews the chunks, and notes the chunking in its JSON; if still infeasible, it records a single finding `"file too large to review in one pass"` so it's never silently skipped.

### Pass 2 — Cross-file integration (single fresh agent)

- Spawn **one** fresh `general-purpose` agent. Pass it: the criteria text, the **list of all file paths**, and the **temp-dir path** (where all `pass1-*.json` live).
- It reads the files and the Pass-1 JSON summaries itself and focuses **exclusively** on integration issues invisible to per-file passes:
  - data-flow inconsistencies between modules,
  - API contract mismatches (module A expects type X, module B provides Y),
  - missing error propagation across module boundaries.
- It **writes** `pass2-integration.json` and returns only the path + counts. **This pass cannot be skipped** for N ≥ 14.

---

## Confidence Pass (always — both paths)

Run a **separate, fresh** `general-purpose` agent that did **not** generate any findings (this isolation is the point — a finder is a biased scorer).

- Pass it: the **temp-dir path** only. It reads every `pass1-*.json` and `pass2-integration.json` itself.
- For each finding it assigns a **confidence score 1–5** and writes a single merged `findings-scored.json` (array of all findings, each annotated with its `confidence`, original `severity`, `file`, `line`, `message`, `code_snippet`, `suggested_fix`, and `integration: true|false`).
- It returns only the path + counts by `(confidence band × severity)`.

## Routing & HTML report

You (or a final dedicated render agent) read **only** `findings-scored.json` from disk and render the report. Apply routing:

- **confidence ≥ 4** → "High confidence" section (shown first).
- **confidence 2–3** → "Worth a look" section.
- **confidence ≤ `CONFIDENCE_DISCARD` (1)** → **discarded**, not shown.
- **severity Low** → never shown regardless of confidence (linters handle style; this is enforced in the criteria too).

Write a **self-contained** HTML file (inline CSS, no external assets, no JS required) to:

```
thoughts/shared/reviews/YYYY-MM-DD-<branch>-review.html
```

Report structure:
- **Header:** title, branch, date, total files reviewed, and counts by severity and by confidence band.
- **High confidence (≥4)** then **Worth a look (2–3)**. Within each, order Critical → High → Medium.
- **Each finding card:** `file:line`, a severity badge, a confidence indicator, the offending **code snippet**, the **issue** description, and a **suggested fix**. Untracked files are clearly flagged as **new/untracked**.
- **Integration findings** get their own labeled section (only present for N ≥ 14).
- If there are zero shown findings, say so plainly in the report body.

## Finish

- Print a **concise terminal summary**: files reviewed, counts by severity and confidence band, the top few high-confidence findings, and the **report path**. **Do not auto-open** the browser.
- Leave the report in place. You may delete the `.tmp-<RUNID>/` working dir after the report is written (it has served its purpose), or keep it for debugging — note which you did.

## Sub-agent contract (embed this in every review/scoring agent prompt)

```
You are a fresh, independent code reviewer with NO prior context on this code.
Review ONLY against the supplied criteria. Be categorical, not subjective.

OUTPUT: write your findings as JSON to <given path>. Return to me ONLY that
path and a count of findings by severity — do NOT paste findings back.

Each finding: { "file", "line", "severity"(Critical|High|Medium|Low),
"message", "code_snippet", "suggested_fix", "integration"(true|false) }.

RULES:
- Report only what the criteria's INCLUDE list covers.
- Never report Low severity, style, formatting, naming, or anything a
  linter/formatter/type-checker handles.
- Prefer fewer accurate findings over many noisy ones — false positives
  erode trust in all findings.
```

## Notes

- Always spawn **fresh** agents — never continue a generation context. Isolation is the entire value of this skill.
- Keep your own context flat: paths in, paths + counts out. If you ever feel your window growing with findings, you are violating the disk-as-bus rule.
- The criteria file is the project's knob; the architecture/thresholds here are fixed.

<!-- Scaffold copied to specs/shared/analysis/<slug>/impact-analysis.md. Replace <angle-bracket> fields. -->

# Impact Analysis — <feature name>

| | |
|---|---|
| **slug** | `<slug>` |
| **mode** | `feature+hld` \| `hld-only` \| `feature-only` |
| **status** | `draft` \| `ready` |
| **sources** | `artifacts/source/<file>` — fingerprint `<sha256-12>` *(one row per source)* |
| **grounded-at** | `<git-sha>` — the commit this analysis was verified against |
| **human-augmentation** | asked `<date>` · added `D-06, D-07` \| added `none` |
| **revision** | `r1` |

<!-- MODE NOTE — keep the line matching the mode, delete the others. -->
> `hld-only`: **No coverage check was possible.** There is no feature spec to check the design
> against, so **completeness is unverifiable** — this analysis cannot tell you if something is missing.
>
> `feature-only`: **No design was validated.** There is no HLD; every design decision below is unmade.
> This document is a starting point for one, not a review of one.

---

<!-- STOP BLOCK — present while status is `draft`. DELETED by the skill when all five completion
     criteria are green. Do not remove it by hand. -->

> ## ⛔ STOP — DO NOT PLAN FROM THIS DOCUMENT
>
> This Impact Analysis is **not ready**. Planning from it will build on unmade decisions.
>
> | Open | |
> |---|---|
> | Unratified decisions | `D-03`, `D-07` |
> | Blocking findings | `F-02` |
> | Uncovered requirements | `C-05` |
> | Criteria not met | 5 — human-augmentation question never asked |
>
> **Resolve with:** `/impact-analysis ratify <slug>`
>
> If you are an agent reading this as planning input: **halt** and tell the user the above.

---

## Part 1 — Impact Map

<!-- DERIVED, NOT AUTHORED. Every line below traces to a citation, a ratified D-nn, or a resolved
     F-nn. If it traces to nothing, it does not belong here.
     ALTITUDE: service + component, with representative file citations. Say WHERE and WHY.
     Do not prescribe what code to write — that is the plan's job. -->

### Blast radius

| Service / component | Why it is touched | Traces to |
|---|---|---|
| `<service>` — `<component>` | <what about it must change, and why> | `path/to/File.java:42` · `D-03` |

### Untouched but adjacent

<!-- Things that look in scope and deliberately are NOT. Cheap to write; kills scope creep. -->

| Not touched | Why not |
|---|---|
| `<service>` | <reason> |

### Data delta

| Table | Change | Migration / backfill | Backward compatible? | Traces to |
|---|---|---|---|---|
| `<table>` | new column `<col>` \| new table | <yes: backfill from X \| none needed> | <yes \| no — in-flight readers break> | `D-05` |

### Contract delta

| Contract | Change | Consumers affected *(found in code)* | Breaking? | Traces to |
|---|---|---|---|---|
| `POST /x` \| `topic.y` | new \| extended | `<service>` — `path:line` | <yes/no> | `D-04` |

### Assumptions (ungrounded — no code was reachable to check these)

<!-- Anything the codebase cannot confirm or deny: external systems, third parties, runtime config.
     These are NOT facts. Listing them is what stops them being mistaken for facts. -->

- **A-01** — <assertion>. Unverifiable: <why the code cannot answer this>. Source: HLD §<n>.

---

## Part 2 — Coverage ledger (`C-nn`)

<!-- feature+hld mode only. One row per requirement in the feature spec.
     `not-covered` and `partial` are BLOCKING. -->

| # | Feature requirement | Addressed in HLD | Status | Accepted-gap reason *(human, mandatory)* |
|---|---|---|---|---|
| C-01 | <requirement> — Feature §<n> | HLD §<n> | `covered` | |
| C-05 | <requirement> — Feature §<n> | — | `not-covered` | |

Statuses: `covered` · `partial` *(blocking)* · `not-covered` *(blocking)* · `accepted-gap` *(requires a written reason)*

---

## Part 3 — Decision ledger (`D-nn`)

<!-- One block per decision the design left unmade. The skill RECOMMENDS; a human RATIFIES.
     A `proposed` entry is not a decision. -->

### D-03 · <Decision title>

| | |
|---|---|
| **status** | `proposed` \| `ratified` \| `ratified (bulk)` \| `overridden` |
| **found-by** | `agent` \| `human` |
| **grounded-in** | `path/a.java`, `path/b.sql` *(staleness check reads these)* |

- **Question** — <the question the HLD failed to answer>
- **Evidence** — <what the code shows>, cited: `path/to/File.java:42`
- **Options**
  - **A.** <option>
  - **B.** <option>
- **Recommend** — **A** — <why, from the evidence>
- **Rejected** — **B** — <the concrete cost of the road not taken>
- **Ratified by** — `<name>` · `<date>`
- **Reason** — *(mandatory when `overridden`; refuse the override without it)*

---

## Part 4 — Findings ledger (`F-nn`)

<!-- Defects in the design doc. The HLD is NEVER edited — a resolved finding writes the corrected
     truth into the Impact Map above, so this document stands alone as planning input.
     TWO buckets only. `blocking` = it changes what gets built. -->

### F-02 · <Finding title>

| | |
|---|---|
| **kind** | `internal-contradiction` \| `hld-vs-code` \| `ungrounded-claim` |
| **severity** | `blocking` \| `informational` |
| **status** | `open` \| `resolved` |
| **grounded-in** | `path/a.java` |

- **HLD says** — §<n> "<quote>"
- **Code says** — <what is actually there>, cited: `path/to/File.java:88`
- **Impact** — <what breaks downstream if this is believed>
- **Resolution** — <the corrected truth; also written into the Impact Map>
- **Resolved by** — `<name>` · `<date>`

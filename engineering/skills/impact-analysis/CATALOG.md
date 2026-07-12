<!-- Scaffold copied to specs/shared/analysis/decision-catalog.md on first use.
     This file is the PRODUCT REPO's knowledge, not the skill's. It is version-controlled beside the
     code it describes, and it grows. Do not pre-fill it with a guessed catalog — a guessed catalog
     encodes the wrong architecture. -->

# Decision catalog

Design decisions this codebase has needed before, and that a design doc is likely to leave unmade.
The `impact-analysis` skill walks these prompts **in addition to** discovering fresh decisions.

**This file starts empty and accretes.** It is appended by `impact-analysis` Step 7 with:

- every decision a **human** had to add (`found-by: human`) — by definition, exactly what the agent
  could not see on its own, and therefore the highest-signal entry here;
- every decision a human **overrode** — the agent being wrong in a way this team knows how to be right.

Empty catalog → pure discovery. Rich catalog → discovery *plus* a checklist. It converges on this
team's real architecture over a handful of features, at a cost of one append per run.

Prune it like any other checklist: an entry that has been `N/A` on every feature for a year is
sediment. Delete it.

---

## Prompts

<!-- One row per learned decision class. `Learned from` is what makes an entry auditable — and
     deletable. -->

| # | Prompt — check whether this feature needs… | Learned from | Added |
|---|---|---|---|
| | | | |

---

## Example of a filled row (delete this section once the table has real entries)

| # | Prompt — check whether this feature needs… | Learned from | Added |
|---|---|---|---|
| 1 | …a new Kafka topic, and who consumes it | `checkout-v2` — human added `D-07`; the agent proposed a sync call and missed that this domain is event-driven | 2026-07-12 |
| 2 | …a tenant-scoping column on any new table | `bulk-export` — human overrode `D-02`; every table here is tenant-scoped by convention, not by schema constraint | 2026-07-19 |

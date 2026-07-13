# dmax-skills

A personal library of skills, subagents, and hooks for the full software engineering lifecycle. Goal: bring harness — predictable, reusable behaviour — to AI coding agents. Built by a tech lead/architect, stack-agnostic, covering spec → implementation → review → ops.

Built for Ravi first; shaped for sharing after battle-testing.

## Authoring skills

Read `.claude/skills/writing-great-skills/SKILL.md` before writing or editing any skill. It is the authoritative reference for vocabulary, principles, and failure modes.

The rules that matter most:
- **Predictability** is the root virtue — same process every run, not same output
- Choose **model-invoked vs user-invoked** before writing; each spends a different budget (context load vs cognitive load)
- **Leading words** anchor behaviour more efficiently than prose
- **Completion criteria** must be checkable — vague gates invite premature completion
- **Prune** without mercy: no-ops, sediment, and duplication cost tokens and clarity

## Existing skills

| Skill | Invocation | Purpose |
|---|---|---|
| `writing-great-skills` | user | Authoritative reference for skill authoring — read before creating |
| `grilling` | model | Relentlessly stress-test a plan or design |
| `grill-me` | user | Alias that invokes `/grilling` |
| `handoff` | user | Compact the current conversation into a handoff doc for a fresh agent |
| `high-agency` | user | Coach a problem up the low-to-high agency spectrum — a tracked case, or `quick` for a one-shot verdict + moves |
| `feature-authoring` | user | Author a consistent, multi-session feature spec from raw ideas/docs in a brownfield workspace |
| `impact-analysis` | user | Reconcile feature/HLD specs against real code; force the skipped design decisions; emit the grounded artifact `create-plan` consumes instead of the HLD |

## Skill pipeline

Raw input (articles, essays, notes, ideas) lives in `specs/personal/resources/` or is provided in conversation.

**Draft → test → graduate:**
1. Draft the skill in `.claude/skills/<name>/SKILL.md` — this makes it immediately invokable
2. Test by running it on real work; iterate until behaviour is predictable
3. Graduating version moves to the appropriate domain folder:
   - `engineering/skills/` — software lifecycle skills
   - `personal/skills/` — skills distilled from personal reading/ideas

The draft stays in `.claude/skills/` throughout development. Only promote when it holds up under real use.

## Subagents and hooks

These are being built out. When adding:
- **Subagents**: define a clear `agentType`, scoped tool set, and the signal that should trigger them
- **Hooks**: prefer narrow shell commands targeting a specific lifecycle event; document the trigger and expected side-effect inline

## Lifecycle coverage

All phases are in scope: spec/planning · implementation · review/quality · deploy/ops. Skills should generalize across stacks — avoid hardcoding languages or frameworks unless the skill is explicitly stack-specific.

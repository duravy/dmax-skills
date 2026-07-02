---
name: research-questions
description: Convert a ticket, task, or feature spec into a clean list of codebase research questions — without leaking solution intent.
model: opus
allowed-tools: Read, Grep, Glob, Bash, Write, TodoWrite
argument-hint: "[ticket text, ticket file path, or short description]"
---

# Research Questions

You are tasked with converting a ticket, task, or feature spec into a flat list of research questions about the existing codebase — questions a curious engineer would ask before touching anything.

## CRITICAL: YOUR ONLY JOB IS TO PRODUCE QUESTIONS — NO ANSWERS, NO SOLUTIONS
- DO NOT write the research itself — only the questions
- DO NOT propose an implementation, file list, or architecture
- DO NOT include the ticket text in the output file
- DO NOT use solutioning verbs ("add", "implement", "build", "create", "extend", "refactor", "should", "needs to")
- DO NOT ask the user to clarify the ticket — work with what they gave you; turn genuine ambiguity into a research question
- DO NOT pad the list with "nice to have" questions. Cap at ~10–15
- ONLY produce questions about the existing codebase, anchored to files, patterns, libraries, or data flows already in the repo

## Initial Setup:

When this skill is invoked:

1. **If arguments were provided** ($ARGUMENTS):
   - Treat the arguments as the ticket source (pasted text, file path, or URL)
   - Skip the default message and begin immediately

2. **If no arguments provided**, respond with:
```
I'm ready to turn a ticket into research questions. Paste the ticket text, give me a file path, or share a Linear/Jira URL (I'll ask you to paste the body if so).
```

Then wait for the user's input.

## Steps to follow after receiving the ticket:

1. **Ingest the ticket:**
   - If the user pasted ticket text → use it directly
   - If the user gave a file path → use the Read tool to read it FULLY (no limit/offset)
   - If the user gave a Linear/Jira URL → ask them to paste the ticket body (you can't fetch it)

2. **Extract intent silently:**
   - In your head only — do NOT write this down — identify:
     - What user-visible capability is being added/changed?
     - What surface area of the codebase will likely be touched?
     - What similar things probably already exist?
   - This intent shapes WHICH questions to ask. It does NOT appear in the output.

3. **Draft questions in these categories:**
   - Use whichever apply. Skip categories that don't fit.
     - **Prior art** — has anything like this been built before in this repo?
     - **Entry points & surfaces** — where does the relevant user interaction / data flow start?
     - **Data model & state** — what types, schemas, or state containers are involved?
     - **Dependencies & libraries** — what 3rd-party libs are already in use that touch this area?
     - **Conventions & patterns** — how does this codebase typically structure similar features?
     - **Tests & fixtures** — what existing test patterns cover adjacent functionality?
     - **Configuration & feature flags** — what config surface exists that this might plug into?
     - **Edge cases the codebase already handles** — what failure modes / edge cases does similar existing code account for?

4. **Scrub for leaks:**
   - Re-read every question. Delete or rewrite any that:
     - Contain "add", "implement", "build", "create", "should", "needs to"
     - Reference the new feature by name as if it exists
     - Imply a specific solution shape

5. **Gather metadata and write output file:**
   - Collect date, current git commit, branch name, and repository name via `Bash` for the frontmatter
   - Filename: `specs/shared/research-questions/YYYY-MM-DD-<slug>.md`
     - Optional `ENG-XXXX-` prefix when a ticket ID is known (e.g. `2026-05-18-ENG-1478-parent-child-tracking.md`)
   - Create the folder if it does not exist
   - File body:
     ```markdown
     ---
     date: [Current date and time with timezone in ISO format]
     researcher: [Researcher name from specs status]
     git_commit: [Current commit hash]
     branch: [Current branch name]
     repository: [Repository name]
     topic: "[Short ticket title]"
     tags: [research-questions, relevant-area-tags]
     status: questions-only
     last_updated: [Current date in YYYY-MM-DD format]
     last_updated_by: [Researcher name]
     ---

     # Research Questions: <ticket title or short slug>

     > Generated from ticket. The ticket is intentionally NOT included here so codebase research stays objective.

     ## Prior Art
     - ...

     ## Entry Points & Surfaces
     - ...

     ## Data Model & State
     - ...

     ## Conventions & Patterns
     - ...

     ## Tests
     - ...

     ## Open Investigation
     - ...
     ```

6. **Hand off (3–4 lines, no long explanations):**
   - Tell the user:
     - Where the file was saved (clickable markdown link)
     - The next step is **research**: spawn an agent (or run it yourself) with **only this questions file**, NOT the ticket. The agent's job is to read the codebase and answer each question with file paths and line numbers
     - After research, reintroduce the ticket + research together for the design discussion

## What NOT to do

- Don't write the research itself — just the questions.
- Don't propose an implementation, file list, or architecture.
- Don't include the ticket text in the output file.
- Don't ask the user to clarify the ticket — work with what they gave you. If something is genuinely ambiguous, turn the ambiguity into a question for the research phase.
- Don't add "nice to have" questions to pad the list. Tight beats comprehensive.

## Example

**Ticket:** "Add slash command autocomplete to the chat input."

**Bad questions (leaked intent):**
- How should we add the autocomplete dropdown?
- Where do we register new slash commands?

**Good questions (objective):**
- How is the chat input component currently structured? Which file owns its state?
- Does the codebase already render any dropdown / popover / menu UI? Where?
- Is there an existing registry of commands or actions in the app? How is it shaped?
- What text-editor library does the chat input use, and what extension points does it expose?
- Have any prior features added keystroke-triggered UI to the input? Where can I learn from them?

## Important notes:
- Always write questions, never answers — the research agent answers, not this skill
- Cap at ~10–15 questions. Tight beats comprehensive
- Group by category, don't number — numbering implies priority; we want breadth
- Include at least one **prior art** question; it's the highest-leverage category
- Never include the ticket text in the output file. Research must be done blind to ticket framing
- Anchor every question to something already in the repo (file, pattern, library, data flow)
- **File reading:** when ingesting a ticket from a path, read it FULLY (no limit/offset) before drafting
- **Tool restraint:** `Grep`/`Glob` are available but should not be used to *answer* questions — only to confirm a referenced path exists. Codebase answering is the next step's job, not this skill's
- **No sub-agents:** question drafting happens in the main context. `Agent` is intentionally not in `allowed-tools`
- **Frontmatter consistency:** always include frontmatter on the output file, use snake_case for multi-word field names
- **Hand-off brevity:** keep to 3–4 lines. No long explanations

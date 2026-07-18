# Implementation Plan

Transform the existing v2/ai-template.md into a self-driving content production system that auto-picks the next unstudied topic from v2/README.md, generates the 4-level Senior answer, tracks progress via checkboxes, and stops after each topic for user approval.

The existing setup has a 4-level answer template (ai-template.md) and 88 topics across 3 lists (JS Core, React/Next, Architecture) with 3 priority levels each. Currently there is no progress tracking, no session continuity, and no automation — each session requires manual topic selection and coordination. The implementation adds a progress checklist file, content output directory, and enriched template instructions so every Claude session begins by reading state, picking the next topic, and executing independently.

[Types]

No new TypeScript/type definitions needed. The progress data model is implicit in the checklist format:

- Topic: `{list_number}. {priority} {title}` — string
- State: `[ ]` (unchecked) or `[x]` (checked) — boolean
- Grouping: by list (1/2/3) then by priority (⭐⭐⭐/⭐⭐/⭐)
- Completion entry: `{date} | {topic} | {list}` — stored in v2/content/INDEX.md

[Files]

Minimal file additions and modifications:

- **NEW: `v2/progress.md`** — Mirror of all 88 topics as checklists grouped by list (JS Core, React/Next, Architecture) and priority level (⭐⭐⭐/⭐⭐/⭐). This is the single source of truth for what remains.
- **NEW: `v2/content/`** — Directory for generated article files.
- **NEW: `v2/content/INDEX.md`** — Date-stamped log of completions: `| Date | Topic | List | File |`
- **MODIFIED: `v2/ai-template.md`** — Add automation instructions block at the top: session startup sequence, workflow rules (stop after each topic, wait for approval, mark progress), and how to read/write progress.md.
- **MODIFIED: `v2/README.md`** — Minor: add a cross-reference line at the top pointing to `v2/progress.md` for tracking.

Files NOT modified: existing topic content files (README.md, HTML/, CSS/, DOM/, JAVASCRIPT/, PWA/, RENDERING/ — those are legacy, untouched).

[Functions]

Two logical operations, not explicit functions:

1. **next-topic selection**: Read `v2/progress.md` line-by-line. Find first line matching `- [ ]` pattern. Extract list number, priority stars, and topic name. Return the full topic entry context (which list it belongs to, which priority level).

2. **progress update**: After user approval, find the matched `- [ ]` line in `v2/progress.md` and replace with `- [x]`. Prepend a new row to `v2/content/INDEX.md` with the completion record.

These are implemented as sed commands embedded in the template instructions (no new scripts).

[Classes]

No classes. All logic is declarative: sed commands for text manipulation, and the 4-level answer generation follows the existing template structure.

[Dependencies]

No new dependencies. Uses only:
- `sed` for text replacement (progress checkbox update, INDEX prepend)
- `grep` for next-topic detection
- Standard Markdown rendering

[Testing]

No formal test framework. Verification via:
- After each topic completion, confirm the checkbox in `v2/progress.md` is marked `[x]`
- Confirm `v2/content/INDEX.md` has the new row
- Confirm `v2/content/{topic-slug}.md` exists and contains the 4-level answer

[Implementation Order]

Steps executed sequentially, one topic at a time (user confirms between each):

1. Create `v2/progress.md` — full checklist of all 88 topics grouped by list and priority, all `[ ]`.
2. Create `v2/content/INDEX.md` — empty table header row.
3. Modify `v2/ai-template.md` — prepend automation instructions block.
4. Modify `v2/README.md` — add cross-reference line to `v2/progress.md`.
5. Generate first topic (JS Core ⭐⭐⭐ #1: Event Loop) → run the 4-level template → present → wait.
6. On confirmation → mark `[x]` in progress.md, append to INDEX.md → STOP.
7. Repeat step 5-6 for each subsequent topic in priority order: JS Core ⭐⭐⭐ (1-8), React ⭐⭐⭐ (1-10), Arch ⭐⭐⭐ (1-10), JS Core ⭐⭐ (9-16), React ⭐⭐ (11-20), Arch ⭐⭐ (11-20), JS Core ⭐ (17-25), React ⭐ (21-28), Arch ⭐ (21-30).
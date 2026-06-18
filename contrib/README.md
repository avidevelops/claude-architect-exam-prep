# Community Contributions

This directory holds community give-backs to
[`claude-architect-exam-prep`](https://github.com/avidevelops/claude-architect-exam-prep),
contributed under the repository's **CC BY 4.0** license.

## `expanded-counter-explanations.json`

A machine-readable, per-option restructuring of the "Why the others are weaker"
reasoning that already lives in the main `README.md`. Each entry is keyed to the
README question number (`Q1` … `Q33`) and carries:

- `correct_option` — the correct answer letter (A–D)
- `why_correct` — the rationale for the correct option
- `per_option_counter_explanations` — an object keyed by the distractor's option
  letter (e.g. `"B"`, `"C"`, `"D"`) so a quiz / flash-card tool can render the
  counter-explanation for each wrong answer programmatically, rather than parsing
  prose
- `exam_takeaway` — the one-line takeaway

This is useful for tools that surface one question at a time (for example the
flash-card / quiz view linked at the bottom of the main README), where having the
per-distractor reasoning available as structured data avoids re-parsing Markdown.

### Why

We ingested this repo's question content into a small non-commercial cohort
training tool (see [Issue #2](https://github.com/avidevelops/claude-architect-exam-prep/issues/2),
where the author kindly granted CC BY 4.0 and invited improvements back as a PR).
This file returns the structured counter-explanations as promised.

### Attribution & license

- **Original question content:** © avidevelops — `Source: avidevelops,
  https://github.com/avidevelops/claude-architect-exam-prep, CC BY 4.0`
- **This enrichment** (restructuring the prose into letter-keyed JSON): MSApps
  Mobile × OpsAgents, also released under **CC BY 4.0**. Changes were made
  (reformatting into machine-readable form); no question text or correct answers
  were altered.

### Notes

- Question **Q29** ("Temporal Differences vs. Data Contradictions") is omitted: it
  has no enumerated answer options in the source, so no per-option mapping applies.
- Hebrew translations of the questions are produced at runtime in our training tool
  and are not yet finalized as a static export; we're happy to follow up with a
  Hebrew (`he`) companion file in a later PR once that batch is signed off, if
  that would be useful to you.

— MSApps Mobile × OpsAgents · michal@opsagents.agency

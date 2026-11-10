---
name: goldfish-drift-report
description: Compare checked-in specs with the current code and report only where they diverge. Works on ODD feature documents (odd/tasks/*.md, checking each S# spec), design docs, or READMEs in a directory; each document is checked by a fresh read-only subagent. Use when the user asks to detect drift, check whether specs still match the code, or audit old feature documents.
license: Apache-2.0
---

# Goldfish drift report

Specs rot. This skill finds where a checked-in document and the current code
disagree, and reports only the differences. The human decides whether the
document or the code is wrong.

Adapted from the Elephant-Goldfish Model `egm-drift-report` skill
(Apache-2.0, Google LLC). See NOTICE for the changes.

## Inputs

- `Target` (required): a directory (for example `odd/tasks/` or
  `docs/designs/`) or a list of files. If missing, ask for it in one plain
  question and stop.
- `Output` (optional): a report file path. Default: report in chat only.
  Write a file only when the user gives a path or asks for one.

## Workflow

1. List the Markdown files in `Target`. If there are more than 10, tell the
   user the count and ask whether to check all of them or a subset; stop and
   wait.
2. For each document, start one read-only subagent (Pi: `subagent_run` with
   `gentle-ai-explore`). Run at most 4 at a time. If no subagent can be
   started, check the documents yourself one at a time and say so in the
   report. Send only:

   ```text
   Read-only drift check. Do not modify any file.
   Document: <absolute path>
   Repository root: <absolute path>

   1. Read the document. List what it says the code does or contains:
      behavior, commands, options, messages, files, symbols, data fields.
      For an ODD feature document, use each S# under "## Specs" as one claim
      and quote its ID; ignore "## Log" except as context.
   2. Find the code for each claim. Use paths the document names; otherwise
      search for the symbols, commands, or messages it mentions.
   3. Read the current code and compare. Report only divergences:
      - Missing: described but absent.
      - Different: present but behaves or is named differently.
      - Shortcut: hard-coded or stubbed where the document describes logic.
      - Architecture: built with a different structure or library.
      - Unverifiable: you could not locate the code; say what you searched.
      - Undocumented: a user-visible command, option, action, or message in
        the same area that the document does not mention. Not internal
        details, extra key aliases, or edge-case guards.
   When the document enumerates a list (actions, commands, options,
   environment variables, fields), an item the code has but the list
   omits is Different, not Undocumented.
   Each divergence: claim (S# or quote), category, evidence, and one line
   on the difference. Evidence is path:line followed by a short verbatim
   fragment of that line; re-open the file to copy it. For Different, quote
   both the document's words and the code's actual text or behavior.
   Output three sections:
   - "Coverage": one line per claim you extracted (every S#, or every
     quoted claim for non-ODD documents): `S1 — match | drift |
     unverifiable — <code location checked>`. Check every sub-clause of a
     claim (each "and", "if", "otherwise" branch), not just its main idea.
     Then check the other direction: open the function that implements the
     claim and read every condition or early return that changes its
     result. A code branch that contradicts the claim (for example it
     refuses where the document says it asks) is Different, even when
     every branch the document names exists.
     Never skip a claim; this section proves what was checked.
   - "Drift" (Missing, Different, Shortcut, Architecture, Unverifiable).
   - "Undocumented" (at most 5, most visible first).
   If nothing diverges, still output Coverage, then exactly:
   No drift detected.
   ```

3. If a subagent fails or returns nothing, retry it once. A second failure is
   listed as `not checked`, never as `No drift detected`.
4. Check the Coverage section: every S# (or claim) in the document must
   appear. If any is missing, send the same subagent one follow-up asking
   it to check only the missing claims; if it still omits them, list them
   as `not checked` in the report.
5. Spot-check one cited `path:line` per document with real findings by
   searching for its quoted fragment. A fragment found a few lines away is
   fine; correct the line number. Drop any finding whose fragment does not
   exist and say you dropped it.

## Report

```markdown
# Drift report — <YYYY-MM-DD>

Checked: <n> · With drift: <n> · Clean: <n> · Not checked: <n>

## <document path>
- [Missing] S3 — "<claim>" — no match for `--dry-run` (searched lib/, bin/)
- [Different] S5 — <claim> — `lib/cli.ts:88` prints "<actual>"
```

Under each document, list Drift items first, then an `Undocumented:`
sub-list. List documents with Drift first, then documents with only
Undocumented items, then `Clean:` as one line of names, then `Not checked:`
with the reason. Present it in the user's language; keep
quotes, paths, and messages verbatim.

## Limits

- One pass is not exhaustive. Each run can miss a different divergence,
  even with full coverage lines. For an audit that matters (before a
  release, or before trusting an old spec), run the report twice and merge
  the results; a divergence found by either run counts.
- This reads code; it does not run it. Behavior that only shows at runtime
  can be `Unverifiable`. Offer to run the spec's examples as a separate,
  explicit step.
- A finished ODD task document describes the feature as built at that time;
  later intended changes may show as drift. Report them; do not guess intent.
- Never edit documents or code from this skill.

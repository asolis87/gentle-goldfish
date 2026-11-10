---
name: goldfish-zoom-out
description: Fresh-context simplicity review of a design, spec, or ODD feature document before implementation. A read-only subagent with no conversation history checks for YAGNI, speculative abstractions, premature optimization, and new code that duplicates something the repository already has, backing every reuse claim with a code search. Use before implementing a large feature, when a plan adds new modules or helpers, or when the user asks whether a design is over-engineered.
license: Apache-2.0
---

# Goldfish zoom-out review

The author of a design is the worst person to ask whether it is needed or
whether it already exists. This skill asks a fresh-context reviewer that
cannot see the discussion and must prove reuse claims in the code.

## Inputs

- `TargetFile` (required): the design or spec, for example
  `odd/tasks/<feature>.md`. If missing, ask for it in one plain question and
  stop.
- `ContextFiles` (optional): extra files the reviewer may read. Never pass
  chat transcripts or your own summary of the intent.

## Dispatch

1. Confirm a subagent tool exists (Pi: `subagent_run`). Use a read-only
   agent; prefer `gentle-ai-explore`. If no subagent can be started, stop and
   tell the user: a review in the same context is not a goldfish review. Do
   not run the persona inline.
2. Send only this task, with the placeholders filled:

   ```text
   Read-only review. First read this persona file completely and follow it:
   <absolute path to references/persona.md in this skill directory>

   TargetFile: <absolute path>
   ContextFiles: <absolute paths, or "none">
   Repository root: <absolute path>

   Do not modify any file. Return only the report format the persona defines.
   ```

3. If the subagent fails, returns nothing, or ignores the format, retry once.
   A second failure is reported as `not run`, never as "no findings".

## After the report

1. Check each reuse claim: open the cited `path:line` yourself. Discard a
   claim whose evidence does not exist and tell the user you discarded it.
2. Show the user, in their language: Blocker and Major findings with the
   proposed simpler alternative, then a count of Nits.
3. The human decides. Scope cuts change requirements: in ODD, record the
   decision verbatim in `## Log` and rewrite only the affected `S#`. Never
   drop a requirement the user stated just because a reviewer called it
   YAGNI; ask.
4. Never downgrade severity. To dismiss a finding, cite evidence the reviewer
   could have read.

## Limits

This is not a correctness, security, or test review. It does not replace
Gentle Shell's 4R lenses, judgment-day, or verification; it runs earlier, on
the plan, where cutting scope is cheapest.

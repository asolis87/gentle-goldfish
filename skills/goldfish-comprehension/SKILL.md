---
name: goldfish-comprehension
description: Fresh-context comprehension review of a spec or ODD feature document (odd/tasks/<feature>.md). A read-only subagent with no conversation history paraphrases what the document asks for and lists undefined terms and assumed knowledge, so the author can compare it with the intent before implementation. Use before delegating or implementing a large tracked feature, or when the user asks whether a spec is clear.
license: Apache-2.0
---

# Goldfish comprehension review

A goldfish is a reviewer that starts in a fresh context. It sees the document
and the repository, never the conversation that produced the document. Its
value is that it cannot fill gaps with what you already know.

## Inputs

- `TargetFile` (required): path to the spec, for example
  `odd/tasks/<feature>.md`. If missing, ask for it in one plain question and
  stop.
- `ContextFiles` (optional): extra files the reviewer may read. Never pass
  chat transcripts, summaries of the discussion, or your own interpretation.

## Dispatch

1. Confirm a subagent tool exists (Pi: `subagent_run`). Use a read-only
   agent; prefer `gentle-ai-explore`. If no subagent can be started, stop and
   tell the user: a review in the same context is not a goldfish review. Do
   not run the persona inline.
2. Send the task below, filling only the placeholders. Do not add background,
   goals, or hints about what the document "means".

   ```text
   Read-only review. First read this persona file completely and follow it:
   <absolute path to references/persona.md in this skill directory>

   TargetFile: <absolute path>
   ContextFiles: <absolute paths, or "none">

   Do not modify any file. Return only the report format the persona defines.
   ```

3. Wait for the report. If the subagent fails, returns nothing, or ignores the
   format, retry once. If it fails again, report the review as `not run`.
   Never present a failed run as a clean review.

## After the report

1. Compare the paraphrase with what the user asked for. For an ODD feature
   document, compare each `S#` paraphrase with the verbatim `S#` text and the
   `L1` request in `## Log`. A wrong paraphrase is a finding, even when no
   term is undefined.
2. Show the user, in their language: the paraphrase mismatches first, then
   Blocker and Major findings, then a count of Nits.
3. Do not edit the document to satisfy the reviewer by yourself. Propose
   concrete wording; the human decides. In ODD, an accepted change appends a
   `## Log` entry and rewrites only the affected `S#`.
4. Never downgrade a finding's severity. To dismiss one, cite evidence the
   reviewer could have read (a file and line); "I know what it means" is not
   evidence, because the next reader will not.

## Limits

This checks whether the document can be understood without its author. It
does not check whether the design is correct, tested, or simple; use
`goldfish-zoom-out` for simplicity and reuse, and normal verification for
correctness.

---
name: opus-prompting
description: Prompting guidance for Claude Opus 5.5 (claude-opus-5-5, anthropic/claude-opus-5-5). Use when writing or revising a prompt, subagent task, system prompt, CLAUDE.md, or agent harness that Opus will run, or when Opus stops mid-run to report, thinks too long, or makes generic frontend designs.
---

# Prompting Claude Opus 5.5

Sources, read 2026-09-25:

- https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-opus-5-5
- https://claude.dev/blog/getting-the-most-out-of-opus-5-5/

Anthropic's tested wording lives in [references/snippets.md](references/snippets.md).
Use it near-verbatim; add only the snippets the task needs. Opus 5 prompts
still work, so prune before you add.

## How Opus 5.5 differs

- Always thinks before replying and decides how much. Effort is the control.
- Strongest on long, multi-step repo work carried through to passing tests, and
  on code review with few false alarms.
- Keeps the user posted. On long tasks, some updates end the turn with text: a
  summary that names the next step without taking it, an offer to continue, or
  a list of non-blocking choices.
- Gets to work quickly. On loosely specified multi-app tasks it can miss context
  the request didn't point to.
- In long chats, may re-examine earlier answers on short follow-ups.
- Falls back on a few default styles for frontend work. "Avoid a generic look"
  just swaps one default for another.
- Reads charts, diagrams, and screenshots accurately without extra tooling.
- Requests to reproduce its internal reasoning in the reply can be declined
  (`reasoning_extraction` refusal).

## Writing a task prompt

1. **Hand over the whole task in one message.** Name the finish line ("the
   tests pass," "every endpoint is migrated") and when to stop and ask. Then
   let it run.
2. **Delete "think carefully" and "think step by step" lines.** For a quick
   answer, write "Answer directly." To change depth, change effort.
3. **Name the stops you want.** Keep going when a step needs no input; stop
   only when blocked or before anything destructive. Keep your own check before
   risky actions.
4. **Fan out large audits and migrations.** Ask for one subagent per unit, an
   evidence check on each report, and one final table.
5. **Keep the task list in a file** on long runs so it survives compaction.
6. **Ask it to mark what it couldn't confirm** and where it looked.
7. **For design work, list the specific styles to avoid,** then iterate on
   what it picked instead.
8. **Attach images; don't retype them.** Ask a specific question about them.
9. **Ask for the finished file** (spreadsheet, doc), not an outline.
10. **Don't ask it to show its reasoning.** Ask for what you need, such as
    "Explain why you chose this approach in three sentences."

## System prompts and CLAUDE.md

- Unattended agents: add the "keep working" paragraph at the end of the system
  prompt from the first request. Leave it out when a human is in the loop.
- Pair programming: ask for a one-line plan up front and a short recap instead.
- Custom report shape, for example "End every run with three headings: Blocked
  on me, Changed, Found."
- Multi-app agents: add the "explore broadly" sentence. Keep untrusted content
  out of the records it searches.
- Chat: remove think-hard lines. Add the "settled answers" sentences if
  follow-ups are slow, but not for long analysis or agentic work.
- Pasted user content: wrap it in `<pasted_content id="…">` tags and add the
  matching system note.
- Team runs: give a time budget or elapsed-time line.

## Harness and API notes

- Effort: default `medium` (Opus 5 was `high`). Test `low` through `max`;
  reserve `xhigh`/`max` for measured gains. Lower effort reduces thinking more
  reliably than prompt text.
- `max_tokens` covers thinking plus reply. 128,000 works well for long agentic
  turns.
- Change effort per message (beta) instead of top-level to keep the cache.
- `thinking: {"type": "disabled"}` is not accepted. Start at `low`, remove
  prompt lines that stood in for thinking, and read replies by block type.
- Treat a text-only `end_turn` as a report, not completion. Keep a checklist; if
  items are open with no blocker, send a continuation naming them. Stop after
  two or three automatic continuations. Wait for background commands and
  subagents before calling it done.
- Progress: set `thinking.display: "updates"` (beta
  `thinking-display-updates-2026-08-18`). After about five silent tool steps,
  append the reminder as a turn-scoped system message; stop after two or three.
- Declare a send-to-user tool from the first request if it must hand over
  verbatim content mid-turn.
- Dense visuals: higher resolution plus a crop tool or a PIL/OpenCV container.
- Refusals: finding vulnerabilities in source code is allowed. Bio safeguards
  match Fable 5.1.

## Check before sending

- Whole task, finish line, and stop conditions are in one message.
- No think-hard or show-your-reasoning lines.
- Autonomy wording matches whether a human is watching.
- Design asks list concrete styles to avoid.

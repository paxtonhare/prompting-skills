---
name: astra-prompting
description: Prompting guidance for GPT-6 Astra (gpt-6-astra, openai-codex/gpt-6-astra). Use when writing or revising a prompt, subagent task, system prompt, AGENTS.md, or skill that Astra will run, or when Astra stops early, asks too many questions, or over-tests.
---

# Prompting GPT-6 Astra

Sources, read 2026-09-25:

- https://developers.openai.com/api/docs/guides/latest-model (GPT-6 model guidance)
- https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra

Tested drop-in wording lives in [references/snippets.md](references/snippets.md).
Use it near-verbatim when a pattern applies. Add only the snippets the task needs.

## How Astra differs

- Follows instructions closely, including ones buried in skills and AGENTS.md.
  Unclear or conflicting guidance can make it pause and block work early.
- More tentative about how far to go. It may stop after a first implementation
  for review, or ask non-blocking questions, where you wanted it to continue.
- Tests and verifies on its own. On small changes it can test more broadly than
  the task needs.
- Has better judgment about boundaries. Strong "ask first" language written for
  older models can make it stop where you would have been happy for it to go on.
- Delegates to subagents less often than many workflows want.
- Tends toward detailed, formatted answers with lists, tables, and recurring
  phrases.

## Writing a task prompt

1. **Define done before it starts.** Name the finish line. If done includes
   running the result, inspecting it, and fixing what fails, say so. Do not ask
   it to stop for review after the first implementation unless you need that
   decision.
2. **Grant the authority it needs.** For work you know is safe, say so and say
   it can proceed without asking at each step. Ask for approval only after it
   has a concrete, reviewable result. See "Initiative" snippets.
3. **State real boundaries plainly.** Keep the ones that matter (destructive,
   irreversible, external). Drop leftover emphasis written to restrain older
   models.
4. **Point to docs by situation.** "Use database.md for schema changes" beats
   "read database.md before every edit."
5. **Don't tell it to test or check its work.** It already does. If the change
   is small, calibrate testing instead. See "Testing" snippets.
6. **Bound exploration.** If you want it to go beyond a first pass, say what to
   explore and where to stop.
7. **Say when to delegate.** If parallel subagents would help, say when and how
   much to use them.
8. **Specify the output.** State the format, length, and style you want. Use the
   prose and anti-slop snippets when you need plain writing.
9. **Rank instructions.** When skills are loaded, say that the user's
   instructions win over a skill's.

## Writing skills and AGENTS.md Astra will read

- Keep skill descriptions short and exact about when they apply. "Use when
  adding or changing a migration" beats "Use when working with databases."
- Make the root SKILL.md a small router to supporting docs and scripts
  (progressive disclosure). Reading a skill costs context.
- Replace step-by-step itineraries with the goal, constraints, and non-obvious
  facts. Over-specific recipes now hurt more than they help.
- Audit AGENTS.md often. Remove "read everything before every edit," reminders
  to run tests, and stale boundary language.
- Repo files also guide other models. Guidance that helps Sol or Luna may
  overconstrain Astra.

## API and harness notes

- Reasoning effort: `none` is not supported; use `low`. Start `minimal` users at
  `low` and compare.
- Use the Responses API for tool calling. Chat Completions works, but not with
  tools on Astra.
- Remove `temperature`, `top_p`, and `top_logprobs` when effort is not `none`.
- Change effort mid-conversation with a `configuration_update` input item. Keep
  request-level `reasoning.effort` unchanged so the prompt cache holds.
- New in GPT-6: async tool calling (`async: true`, return by `call_id`) and
  mid-turn steering over WebSocket.
- Coming from GPT-5.5 or earlier: replace `prompt_cache_retention` with
  `prompt_cache_options.ttl: "30m"`.

## Check before sending

- Done is defined, including run, inspect, and fix when relevant.
- Safe actions are authorized; real boundaries are stated once, plainly.
- No blanket "read these files first" or "run all tests" lines.
- Output format and delegation expectations are explicit.

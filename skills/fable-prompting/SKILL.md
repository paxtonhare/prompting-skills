---
name: fable-prompting
description: Prompting guidance for Claude Fable 5.1 (claude-fable-5-1, anthropic/claude-fable-5-1, also Mythos 5.1). Use when writing or revising a prompt, subagent task, system prompt, or agent harness that Fable will run, or when Fable stops early, widens scope, rewrites whole files, or goes quiet.
---

# Prompting Claude Fable 5.1

Source, read 2026-09-25:
https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-1

Anthropic's tested wording lives in [references/snippets.md](references/snippets.md).
Use it near-verbatim; add only the snippets the task needs. Fable 5 prompts
still work, so prune before you add.

## How Fable 5.1 differs

- Runs very long tasks with little methodology guidance when the goal is clear.
- Can end the turn early: announces "Next, I'll…" instead of doing it, or asks
  "Shall I apply this?" for work already requested.
- Delivers more than asked: fixes nearby code, extends behavior, commits extra
  test files.
- Rewrites whole files for small changes.
- Writes fewer progress updates during long tool chains, more so at high effort.
- Issues one tool call per turn in coding and computer-use loops when the next
  calls are implied rather than named.
- Prose runs denser; chat replies use less structure than older models.
- Reproduces source wording in summaries without marking it as a quote.
- At `low` effort, searches less and answers from memory more.
- At `xhigh` and `max`, may draft a long deliverable in thinking, then again in
  the reply.
- Safety classifiers can return `stop_reason: "refusal"` on benign coding work.

## Writing a task prompt

1. **Give the goal and done condition.** A clear goal matters more than a
   method. Skip step-by-step recipes.
2. **If it will run unattended, say so.** Use the "Finish the whole task"
   snippet. Its first sentence (the user is not watching) carries most of the
   effect; keep it as written. List any confirmations you still want right
   after it.
3. **Fix the scope.** Use "Delivering work" for open-ended tasks and "Keep
   changes and tests to the task" for coding. Unrequested fixes become
   follow-ups in the summary.
4. **Ask for targeted edits** on code or long files.
5. **Name the updates you want** (one-line intent up front, short recap at the
   end) for human-in-the-loop work. Remove any "hold findings for the final
   response" lines.
6. **For prose deliverables,** add "Please remove all mannered prose." For chat,
   replace blanket anti-formatting rules with the formatting rule snippet.
7. **Avoid refusal triggers.** Ask "Are there any bugs in this program?" rather
   than "Does this compile without errors?" Give docs for lesser-known
   languages. Keep base64 out of tool output.
8. **Push searching when names may be new or fast-moving,** especially at low
   effort.

## Harness and API notes

- Effort: default `high`. Sweep `low` through `max` on your own evals; names
  don't map across models. `medium` roughly matches Fable 5 at lower cost.
- Progress: set `thinking.display: "updates"` (beta
  `thinking-display-updates-2026-08-18`) or updates arrive empty. If your UI
  hides tool output, tell the model in a turn-scoped system message.
- Batching: after each tool-result turn, append the batch nudge as a
  turn-scoped system message (`clear_at: "next_user_message"`, beta
  `mid-conversation-system-clear-at-2026-08-21`). Leave earlier copies in place:
  the API clears them at the next user message and they cost no input tokens,
  while deleting them breaks the cache and thinking blocks.
- History must be append-only. Return assistant turns exactly, thinking
  included. Don't edit earlier turns, the system prompt, or tools mid-session;
  use mid-conversation system messages. Editing a prefix can return a 400.
- Client compaction: replace the whole history with one summary plus the new
  user turn, using the compaction snippet. Try compacting later than before.
- Subagents: have the spawn tool return immediately, deliver results in a later
  user message, and give the lead a separate wait tool.
- Vision: give a crop-and-zoom tool or a container with PIL/OpenCV.
- `xhigh`/`max` long outputs: set `max_tokens` for thinking plus reply, and
  append the long-output note.

## Check before sending

- Goal and done are clear; methodology is light.
- Autonomy and scope snippets match whether a human is watching.
- No compile-check phrasing or anti-narration lines.

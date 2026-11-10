---
name: fable-prompting
description: Prompting guidance for Claude Fable 5.2 (claude-fable-5-2, anthropic/claude-fable-5-2, also Mythos 5.2). Use when writing or revising a prompt, subagent task, system prompt, or agent harness that Fable will run, or when Fable stops early, widens scope, rewrites whole files, or goes quiet.
---

# Prompting Claude Fable 5.2

Source, read 2026-11-10:
https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-2

Anthropic's tested wording lives in [references/snippets.md](references/snippets.md).
Use it verbatim as a starting point; add only the snippets the task needs.
Prune inherited instructions before adding more.

## How Fable 5.2 differs

- Can stop with doable work left, especially at `low` and `medium` effort.
- Without ownership language, tends to add to existing structure and report
  nearby bugs instead of fixing them. Treating code as its own encourages
  consolidation and nearby fixes, but also unrequested changes and higher cost.
- Follows approval rules and accumulated cautions closely; broad prohibitions
  can block routine work even after a separate approval.
- May go quiet during tool chains; progress updates are `thinking` blocks,
  empty at the default `thinking.display: "omitted"`.
- Delegation depends on the harness. Prompts inviting subagents can cause
  substantial use; at `xhigh`/`max`, it may delegate reviews of its own work.
- At `high` and above, thinks before almost every reply, even greetings.
  "Answer directly without deliberating" does not stop thinking at `medium`+.
- Follows style rules closely; rigid sentence limits can make reports choppy.
- Safety classifiers return categorized refusals; Mythos 5.2 has the same
  capabilities, available only to organizations in the verification programs.

## Writing a task prompt

1. **Give the goal and done condition.** Add the "Finishing work" block when
   turns stop early, especially for agentic work at `low`/`medium`. Distinguish
   ordinary errors from deliberate blockers; keep confirmation for risky work.
2. **Name exact approvals.** Reply with the action approved, not "continue."
   Scope approval rules to exact actions and put exceptions inside the rule.
   Name routine, safely repeatable steps that need no approval.
3. **Choose ownership deliberately.** Use the ownership sentence only when
   structural improvements are wanted. Omit it for minimal changes to user-owned
   code; a separate scope rule does not cancel it. Read the measured caveats
   in the reference and review the diff, not just the final summary.
4. **Ask for updates when a human watches.** Use the opening/update/recap block
   and, if needed, the harness reminder. Without harness support, use the
   every-few-tool-calls sentence. Expect some gaps even with both measures.
5. **Audit delegation instructions first.** Only then change the spawn tool's
   description if overdelegation persists. Put cost first, then when delegation
   pays off and how to brief the subagent; check its default model and cost.
6. **Make reports self-contained.** Add "Communicating clearly" after removing
   unnecessary style constraints. Ask for any specific details or length needed.
7. **Keep reasoning in thinking blocks.** Remove instructions to reproduce
   internal reasoning in replies; use `display: "summarized"` to inspect it.

## Harness and API notes

- Effort: explicitly start at `high` (API default); evaluate `medium` and
  `xhigh`. `low`: quick, well-specified, high-volume work, including fast
  subagents; `medium`: interactive coding/chat; `high`: hard coding, research,
  long agentic runs. Most capability
  gains are present at `high`; above it, gains are uneven. Reserve `xhigh` for
  measured gains and `max` for ceiling tests or time/token-insensitive problems.
  Rerun sweeps rather than carrying over Fable 5.1 or Opus 5.5 settings.
- Effort controls thinking, latency, tokens, and tool calls, not reply length;
  use prompts for length/tone. Route easy turns lower instead of saying think less.
- Per-message effort changes preserve cache (beta
  `mid-conversation-output-config-2026-07-01`, platform-dependent) and persist
  until replaced. Changing top-level `effort` next request restarts the cache.
- Progress: set `thinking.display: "updates"` (beta
  `thinking-display-updates-2026-08-18`) or `"summarized"`; render non-empty
  `thinking` blocks between tool calls, not only `text` blocks.
- Count consecutive steps with neither `text` nor non-empty `thinking`, with
  display enabled. After several, append the silence reminder after the latest
  tool results as a turn-scoped system message (`clear_at: "next_user_message"`,
  beta `mid-conversation-system-clear-at-2026-08-21`, where supported).
  Leave every reminder unchanged in later `messages`; do not delete old copies.
- Preserved thinking binds blocks to the prefix. Apart from context editing and
  compaction, keep history append-only: changes to system, tools, or earlier
  messages invalidate subsequent thinking. Messages API rejects by default;
  Message Batches drops invalid blocks. Compaction must retain no earlier turns
  after its summary. See the reference before using `"drop_block"` as a fallback.
- Refusals: inspect `stop_reason: "refusal"` and `stop_details.category`:
  `cyber`, `bio`, `frontier_llm`, `reasoning_extraction`, `general_harms`.
  Benign cyber/bio work can trigger safeguards; use official verification and
  fallback documentation, not bypass prompts. See reference links.

## Migration removal checklist

- Read inherited system prompts, skills, agent files, memories, notes, and
  harness text. Remove unnecessary standing delegation instructions first.
- Test without large prompt-written token budgets and remaining-token counters;
  this is not guidance about the API task-budgets beta, which was not tested.
- Replace prefix-editing harness code with append-only history, context editing,
  or compaction; measure costs before accepting dropped thinking blocks.
- Remove Opus 5.5 "treat that answer as done" / focus-on-new-message lines where
  self-correction matters. They reduce follow-up thinking but suppress correction
  of earlier mistakes; elsewhere, test self-correction before adopting them.
- Remove think-less lines; lower effort instead. Remove rigid, unnecessary prose
  rules before adding the report block.
- Delete obsolete cautions; rewrite absolutes with scope, reason, and approval
  exceptions inside the rule. Audit reviewer conditions and inter-agent notes.
  If stuck, restart with trimmed memory/notes and keep the originals.

## Check before sending

- Run the longest tasks; note early stops, unnecessary approvals, silence,
  excess delegation, and additive code where restructuring was wanted.
- Apply one behavior-specific change at a time; measure quality and token cost.
- Confirmations still protect risky/irreversible actions; deliberate blockers
  remain off limits. Do not work around a tool refusal.

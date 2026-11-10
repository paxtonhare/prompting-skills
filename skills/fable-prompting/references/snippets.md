# Claude Fable 5.2 drop-in snippets

Verbatim blocks from Anthropic's "Prompting Claude Fable 5.2" guide, read
2026-11-10:
https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5-2

The surrounding notes summarize the guide; they are not drop-in quotations.
Add only what observed behavior calls for and measure quality and token cost.

## Finish the whole task

Add to the system prompt if turns end with doable work left, especially at
`low` or `medium`. Preserve the distinction between ordinary obstacles and
intentional restrictions. This does not authorize bypassing a tool refusal.

```text
# Finishing work
Please avoid stopping while you still owe work the user asked for. Be persistent and do not stop unless needed.

While status notes are welcome, as is clarifying with the user on open decisions, carry on with work that does not depend on the user's answer without unnecessary stops.

If you notice yourself inviting the user to redirect you or offering to wait, instead proceed on the next part of the task. You may meet ordinary obstacles like errors, timeouts, locked files, empty results or failing tools. Diagnose them first, and when they are not real blockers, work through them with the access you have (wait and retry, fix the request, use another tool or source) rather than stopping unnecessarily.

If someone clearly decided something is a hard blocker, such as a file marked not to be touched, access that was intentionally withheld, or a safety guardrail, leave it alone, say plainly what you found, and look for another safe way to finish the task. Stopping before the task is complete is very rarely merited, as in when the task can't move forward without user input, or where the blocker is deliberate and should not be worked past. This does not override the need for confirmation on risky or destructive actions.
```

In replays of recorded agentic coding sessions at `medium`, a version of this
text roughly halved sessions ending with doable work left. It removed most
permission/offer-to-continue endings, but not endings listing remaining work.
Output tokens rose about 30%. Keep your own confirmation step for risky or
irreversible actions because the model asks less often.

## Approvals and rule exceptions

Approve the exact action, not a bare "continue," "go," or thumbs-up. The
source's example is “Yes, run the migration on the staging database”. Name the
exact actions approval rules cover. In simulated tasks with nobody to approve,
a rule requiring approval for “sending messages to people outside this
conversation” held back roughly half of routine requests to colleagues/teams.

Put the exception inside the rule: require a go-ahead first rather than an
absolute "never" plus a separate permission. In simulated tests, an already
approved step happened about one time in ten under a "never" rule; a separate
approval line barely changed this. Writing the go-ahead into the rule raised
execution to about nine times in ten. Keep confirmation for risky actions.

Remove obsolete cautions from memories, notes, and agent handoffs. A stale
never-retry note among a dozen notes lowered retries after a busy tool from
nearly all attempts to about four in ten. Explicitly listing safe retries as
approval-free raised retries from almost none to about half in another setup;
a general autonomy instruction did not help. Name retries only for safe-to-repeat
calls. A reviewer's "stop at the first error" still suppressed retries in about
a third of attempts despite a retry allowance; Fable 5.1 and Opus 5.5 retried.

## Say who owns the code

Only use when you want the model to improve structure and make related fixes:

```text
This codebase is yours: what it's for and how it's built are your call, and you'll keep building on it.
```

Measured caveats from small coding tasks:

- Without ownership, or with code presented as someone else's, the model tended
  to add to existing structure, duplicate code, and report nearby issues rather
  than fix them. With this sentence it merged duplicates in about half of tries,
  versus under a tenth without it. Rates varied by task; the result held inside
  a long system prompt. The requested feature worked in nearly every try either way.
- Output tokens increased about 1.5 times. It made several unrequested changes,
  many of them new tests and documentation.
- Apart from bug fixes, existing feature behavior changed in one try in six to
  ten, more often at higher effort.
- Closing summaries named larger changes but often omitted small ones. Review
  the diff, not only the summary.
- Omit this sentence for user-owned code requiring minimal edits. A separate
  scope rule does not cancel it: even with “No drive-by refactors, renames or
  cleanups” earlier in the prompt, duplicates were merged in about a third of tries.

## Progress updates (human-in-the-loop)

First ensure the client renders updates: they arrive as progress-update
`thinking` blocks, empty at default `thinking.display: "omitted"`. Set
`display: "updates"` (beta `thinking-display-updates-2026-08-18`) for update
summaries or `"summarized"` for updates plus summarized reasoning. Render the
non-empty blocks between tool calls, not just `text`.

### Harness silence reminder

Count consecutive tool-calling steps with no `text` and no non-empty `thinking`
block, using `"updates"` or `"summarized"`, not `"omitted"`. After several silent
steps, append this after the latest tool results as a turn-scoped system message
where supported: `clear_at: "next_user_message"`, beta
`mid-conversation-system-clear-at-2026-08-21`.

```text
The user hasn't heard from you in a while: say in a few words what you're doing, then continue.
```

In multi-turn coding tests this roughly halved the longest silent stretch,
without noticeable extra tokens or time. Leave each reminder unchanged in
`messages` on later requests. Removing or rewriting it changes an earlier turn
and invalidates subsequent thinking blocks.

### Opening, updates, and closing recap

Add to the system prompt where a human directly watches the work:

```text
Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own (what you found, what you did, and what's next), so a reader who only sees the last message has the full picture.
```

In coding tests the model spoke before its first tool call every time; output
tokens per task rose roughly 20%. Pair with the harness reminder for the most
updates. Even together, expect runs of several tool calls between updates;
these gaps matter less for asynchronous tasks.

### Without harness support

If the harness cannot change or the platform lacks mid-conversation system
messages, add this sentence to the system prompt:

```text
Every few tool calls, give the user a one-line update on where things stand.
```

This cut most long silent stretches in workflow automation tests.

## Subagents: cost first

The guide does not supply a verbatim spawn-tool description. Review delegation
instructions in system prompts, skills, and agent files before changing the tool
text. A plain tool description with no other delegation instructions probably
needs no change. If overdelegation persists, write a description in this order:

1. What starting a subagent adds to session cost in this harness.
2. When delegation is worthwhile and when it is not.
3. How to brief a subagent that is started.

The tested cost-first description reduced delegation about a third on short
tasks versus a description mostly listing reasons to delegate, with the same
task success. Reduce cost emphasis if delegation is too rare; increase it and
test if too frequent. This is most useful where the harness already invites
delegation and subagents dominate token usage. Short tasks with a minimal spawn
tool rarely triggered delegation; coding-product prompts invited it readily.
At `xhigh`/`max`, Fable 5.2 also used subagents for self-review where Fable 5.1
did not on the same tasks. Parallel work can be faster, and early testers liked
5.2 as orchestrator for other models, but check the default subagent model and
its share of session cost.

## Make reports easy to follow

Add to the system prompt for final reports and handoffs. Remove unnecessary
rigid style rules first; adapt to readers and specify length/details as needed.

```text
# Communicating clearly
Write for an intelligent but low-context reader. The person reading has not seen your tool calls, your reasoning or the files you opened, and may be switching in from other work. Messages should generally be self-contained: the reader should understand what you mean without having to dig for other context. Put the key points in the first few sentences. Practice progressive disclosure in your sentence order, and do not use undefined references and ideas in line 1 that won't make sense until the reader reaches line 12.

Use plain words, and write in clear, understandable full sentences. Do not use a term or abbreviation that you made up during the work. Say what the thing is in a way any reader would understand. Write mostly in the style of ASD-STE100 Simplified Technical English: short sentences, one idea per sentence, active voice, and consistently using the same word for the same thing. Use a paragraph of flowing prose where ideas connect, and keep bullets for lists of separate items. Leave out unnecessary details unless the reader asks and practice concision, but do ensure that your messages have the key context. Aim to make your user-facing text as human-readable and clear as possible.
```

In agentic tests, reports became less dense with fewer semicolons; on workflow
automation tasks, sentences also became shorter.

## Migration: preserved thinking and prompt cleanup

Keep history append-only except for supported context editing and compaction.
Changing system prompts, tools, or earlier messages invalidates subsequent
thinking blocks. By default Messages API rejects the request; Message Batches
drops invalid blocks. Use context editing, or compaction retaining no earlier
turns after the summary, to shorten history safely.

To make Messages API drop invalid blocks too, set
`thinking.block_binding.prefix_mismatch_behavior: "drop_block"` with beta
`thinking-binding-controls-2026-08-01`. This is not a free fix: dropping earlier
thinking every turn caused rethinking and higher output-token use in tests.
That was the worst case; cost depends on how often prefixes change. Measure on
your tasks before relying on it. Append-only history also preserves cache hits.

- [Preserved thinking mismatch behavior](https://platform.claude.com/docs/en/build-with-claude/preserved-thinking#mismatch-behavior)
- [Avoid prefix edits](https://platform.claude.com/docs/en/build-with-claude/preserved-thinking#replace-prefix-edits)
- [Turn-scoped reminders](https://platform.claude.com/docs/en/build-with-claude/preserved-thinking#per-turn-reminders)
- [Context editing](https://platform.claude.com/docs/en/build-with-claude/context-editing)
- [Compaction](https://platform.claude.com/docs/en/build-with-claude/compaction)

Test removal of large prompt-written task budgets and remaining-token counters:
on coding tasks at `high`/`xhigh`, removing a large budget lowered cost with
equivalent performance. This finding does not cover the API's task-budgets beta.

Remove inherited Opus 5.5 chat instructions to treat an earlier answer as done
and focus thinking on the new message where re-examining earlier answers matters.
At `high`, these reduced follow-up thinking but made the model far less likely
to flag its earlier mistakes. Elsewhere, test self-correction before adoption.
Remove instructions to think less: at `medium`+ they did not stop deliberation.
Lower effort instead. Prune standing delegation, rigid prose rules, and stale
cautions; write scope, reason, and exceptions into absolute rules. If restarting
with trimmed memories/notes, keep a copy of the originals.

## Safeguard refusals

Inspect `stop_reason: "refusal"` and `stop_details.category`:

- `cyber`: potential cyber harm, such as malware or exploit development.
- `bio`: potential biological harm, such as dangerous lab methods.
- `frontier_llm`: development of competing AI models.
- `reasoning_extraction`: reproducing internal reasoning in response text.
- `general_harms`: other usage-policy areas.

Benign cyber and life-sciences work can trigger classifiers. For legitimate
organizational work, consult the [Cyber Verification Program](https://support.claude.com/en/articles/14604842)
and [Life Sciences Verification Program](https://www.anthropic.com/news/life-sciences-verification-program).
[Claude Mythos 5.2](https://platform.claude.com/docs/en/models/mythos-5-2/overview)
offers the same capabilities only to organizations verified through these programs.
Remove prompts, skills, and tool descriptions asking for internal reasoning in
the reply. Use `display: "summarized"` and read thinking blocks instead.
Do not bypass refusals or safety guardrails.

- [Keep reasoning in thinking blocks](https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback#keep-reasoning-in-thinking-blocks)
- [Refusals, fallback, and billing](https://platform.claude.com/docs/en/models/fable-5-2/whats-new-fable-5-2#refusals-fallback-and-billing)

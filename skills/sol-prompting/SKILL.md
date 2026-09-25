---
name: sol-prompting
description: Prompting guidance for GPT-6 Sol (gpt-6-sol, openai-codex/gpt-6-sol). Use when writing or revising a prompt, subagent task, system prompt, AGENTS.md, or skill that Sol will run, or when choosing between Sol and Astra for a task.
---

# Prompting GPT-6 Sol

Sources, read 2026-09-25:

- https://developers.openai.com/api/docs/guides/latest-model (GPT-6 model guidance)
- https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra

## What the sources do and don't say

OpenAI publishes no Sol-specific behavior notes. The GPT-6 prompting section was
written from observed GPT-6 Astra behavior and offered "as a starting point
across the GPT-6 model family," to be evaluated per model and workload. So:

- **Facts about Sol:** positioning, reasoning-effort support, API constraints
  (below).
- **Inference:** the Astra-specific fixes (anti-tentativeness, fewer questions,
  less testing) may not apply to Sol. Add one only after you see the matching
  failure on Sol.
- The Astra blog's praise of "GPT-5.6 Sol" continuing for long stretches refers
  to the previous generation. Do not assume it describes GPT-6 Sol.

## When to pick Sol

- `gpt-6-astra`: highest capability; long multi-step work across code, browsers,
  and professional tools.
- `gpt-6-sol`: strong reasoning on demanding tasks, at lower cost and latency.
- `gpt-6-luna`: efficient, repeatable work at scale.

Choose on the reasoning the task needs, latency, and cost.

## Writing a task prompt

Sol has less documented judgment than Astra, so be explicit rather than terse:

1. **State the goal and why.** One or two sentences of intent.
2. **Define done.** The finish line, including running and checking the result
   when that matters.
3. **Give constraints and authority.** What it may do without asking, and the
   few real boundaries (destructive, irreversible, external).
4. **Say how to verify.** Name the command or check. Keep it proportional to the
   change.
5. **Specify the output.** Format, length, and what to report back.
6. **Point to context by situation.** "Use X for Y," not "read everything
   first."

If Sol then shows an Astra-like failure, pull the matching snippet from
[references/snippets.md](references/snippets.md):

| Symptom | Snippet |
|---|---|
| Stops to ask, offers a plan, or quits after a first pass | Initiative and follow-through |
| A skill makes it pause or diverge | Instruction priority |
| Over-formatted or slop-heavy writing | Writing style |
| Over-tests small changes | Testing and verification |
| Under-delegates in a multi-agent harness | Subagent delegation |

## Skills and AGENTS.md shared with Astra

Repo instructions guide every model that reads them. Extra scaffolding that
helps Sol may overconstrain Astra. Put Sol-only guidance in the task prompt, not
in shared AGENTS.md or skills. Keep shared skill descriptions short and precise
about when they apply.

## API and harness notes

- Sol supports `none` reasoning effort (Astra does not). If migrating from
  `minimal`, start at `low` and compare.
- Function calling in Chat Completions works only with
  `reasoning_effort: "none"`. Use the Responses API for reasoning with tools.
- Remove `temperature`, `top_p`, and `top_logprobs` when effort is not `none`.
- Change effort mid-conversation with a `configuration_update` input item;
  keep request-level `reasoning.effort` fixed to preserve the prompt cache.
- GPT-6 family features: async tool calling, mid-turn steering, prompt caching
  with `prompt_cache_options.ttl`.

## Check before sending

- Goal, done, authority, verification, and output are all stated.
- Astra-specific fixes are included only for failures you have seen on Sol.
- Sol-only scaffolding stays out of shared repo instructions.

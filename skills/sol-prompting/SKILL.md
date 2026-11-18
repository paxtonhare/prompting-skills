---
name: sol-prompting
description: Prompting guidance for GPT-6.2 Sol (gpt-6.2-sol), with legacy GPT-6 Sol caveats. Use when writing or revising a task prompt, subagent brief, system prompt, AGENTS.md, or skill that Sol will run, or when choosing between Sol and Astra.
---

# Prompting GPT-6.2 Sol

[OpenAI's GPT-6.2 guide](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-6.2-astra)
(read 2026-11-18) recommends Sol for complex work at lower cost and Astra for
the most demanding tasks. Choose using your workload's quality, latency, and
cost measurements.

## What applies to Sol

OpenAI says the prompt examples address behavior seen when working with Astra
and Sol and offers them as starting points for the GPT-6.2 family. The named
behavior notes (including concise writing and pausing on ambiguity) describe
Astra; the guide doesn't separately characterize Sol's defaults. Evaluate on
Sol before borrowing a remedy for a symptom you saw on Astra. The [older Astra
skills article](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra)
discusses GPT-5.6 Sol's sustained follow-through; that is not evidence about
GPT-6 or GPT-6.2 Sol.

## Write a task prompt

1. **Give the goal and finish line.** State why the outcome matters, what must
   work, and what to inspect or run before stopping.
2. **Give scope and authority.** Allow reversible assumptions within scope,
   report material ones, and continue independent authorized work. Pause at
   actual blockers, material scope decisions, or genuine approval gates, such
   as destructive actions or unapproved external writes.
3. **Name project conventions.** Reuse existing behavior where correct and keep
   code readable. Run relevant checks and add tests when requested, required, or
   needed for a concrete regression that current tests would miss.
4. **Write for the reader.** Specify format, audience, and necessary detail,
   rather than a blanket brevity rule. Ask for a self-contained final covering
   outcome, evidence, and limits without requiring prior commentary.
5. **Point to context by situation.** Say which docs apply to which changes.

See [snippets](references/snippets.md) for short excerpts and untested examples.
For shared skills and repo instructions, keep task-specific scaffolding in the
brief. Flag contradictory owner policy rather than overriding it. Scoped
permission does not supersede higher-priority instructions or repo gates.

## Responses API and migration

- Put stable, reusable guidance in a `developer` message as the **first** item
  in `input`. GPT-6.2 appends top-level `instructions` after `input`; use that
  field only for guidance specific to the current request.
- [Keep the prefix unchanged](https://developers.openai.com/api/docs/guides/keep-prefix-unchanged)
  as settings change. Editing earlier developer messages or tool calls can drop
  reasoning items and make the edited prefix ineligible for cache reuse. Certain
  model changes (including downgrades), or a change of end user or organization,
  can also drop reasoning items.
- Start a new integration at `medium`; migrate `none` or `minimal` to `low`.
  Use Responses for tool calling. Omit `temperature`, `top_p`, and `top_logprobs`;
  omit `logprobs` in Chat Completions and remove `message.output_text.logprobs`
  from Responses `include`.
- From GPT-5.5 or earlier, migrate `prompt_cache_retention` to
  `prompt_cache_options.ttl` set to `"30m"`; review the
  [cache boundaries and billing](https://developers.openai.com/api/docs/guides/prompt-caching#summary-of-model-differences).
- Earlier GPT-6 Sol supported `none` and limited Chat Completions function
  calling to that effort. Keep that and older `configuration_update` recipes
  version-specific; do not assume those rules apply to GPT-6.2.

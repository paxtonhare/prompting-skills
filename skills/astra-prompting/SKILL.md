---
name: astra-prompting
description: Prompting guidance for GPT-6.2 Astra (gpt-6.2-astra), with legacy GPT-6 Astra caveats. Use when writing or revising a task prompt, subagent brief, system prompt, AGENTS.md, or skill that Astra will run.
---

# Prompting GPT-6.2 Astra

[OpenAI's GPT-6.2 guide](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-6.2-astra)
(read 2026-11-18) recommends Astra for the most demanding tasks and Sol for
complex work at lower cost. Its Astra behavior notes are vendor observations;
its prompt patterns are starting points for **both** models, not guarantees.
See [snippets](references/snippets.md) for short excerpts and untested examples.

## What changed for Astra

- **Writing is already concise and direct.** Revisit blanket "be concise"
  instructions. Specify the audience, deliverable, and evidence the reader needs.
- **Ambiguity can still stall work.** Astra may propose a next step and wait.
  Define the finish line and allow reasonable, reversible assumptions inside the
  authorized scope. Keep real approval gates.
- **Describe project conventions.** OpenAI reports improved coding quality and
  efficiency; name the local patterns and caller behavior the change must preserve.
  For visual work, specify imagery, typography, and a reference if available.

## Write the brief around the outcome

1. **Define done.** Name what must work, what to inspect or run, and what to fix
   and recheck before stopping. A plan is not the deliverable unless requested.
2. **Separate assumptions from authority.** Let it proceed under ordinary,
   reversible assumptions and report material ones. Pause at an actual blocker,
   a material scope decision, or an action that needs approval (e.g. destructive,
   irreversible, costly, or external). Continue independent authorized work.
3. **Name conventions and verification.** Reuse existing code where it preserves
   behavior, keep changes readable, and run relevant checks. Add tests when
   requested, required by the repo, or needed for a concrete regression the
   existing suite would miss. Avoid tests that just mirror the implementation.
4. **Require a self-contained final.** State the outcome, evidence, and any
   limits or blockers in terms the reader understands without progress updates,
   tool traces, or subagent shorthand.

## Shared instructions and earlier versions

Keep skill descriptions precise and point to supporting material by situation.
Audit briefs for conflicting finish lines, approval language, and verbosity
rules. Flag conflicts in owner policy rather than silently overriding them.
Scoped permission does not supersede higher-priority instructions or repo gates.

The [earlier GPT-6 Astra skills article](https://developers.openai.com/blog/rethinking-skills-and-prompts-for-gpt-6-astra)
helps diagnose older prompts: dense skill or repo rules could cause early stops,
and testing or delegation sometimes needed calibration. Treat those observations
as legacy context, not GPT-6.2 defaults. If delegation is useful, state when it
is permitted and what the parent must integrate and verify.

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
  Keep older `configuration_update` recipes version-specific; check the current
  API guide before applying them to a 6.2 integration.

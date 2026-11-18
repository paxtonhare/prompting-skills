# Repository workflow

- Keep files small and modular: one job per file, about 800 lines or 50 KB at
  most. Reuse existing patterns.
- Keep each PR to one area and about 500 changed lines, excluding tests and
  pure moves. Split larger work and write `Part of #N` while work remains.
- Push each commit. For a bug fix, first show a regression test failing without
  the fix, then passing with it.
- Check Norris starts after about 10 quiet minutes. Never push empty commits to
  trigger it. Its status is green only at 5/5. In a PR comment, use
  `@checknorrisbot review` for an unreviewed head, `@checknorrisbot re-review`
  after a capped or failed review, or `@checknorrisbot rescore` after replies.
- Answer every finding once in its thread, with `@checknorrisbot` first for its
  findings. Fix real P0/P1 findings with a test and reply `Fixed in <sha> (<change>)`;
  explain false positives with file and line evidence. P0 cannot be risk-accepted;
  only Paxton can accept P1 risk.
- Fix small P2s. Defer the rest in one follow-up issue per PR and reply
  `Deferred to #N: <why minor>`. Answer every P2 before pushing again. Resolve
  Check Norris threads after its verdict accepts the reply or confirms the fix;
  never resolve an unanswered or rejected finding.
- Merge only with Check Norris green on the exact current head, all other checks
  green, and every thread resolved. Use `--match-head-commit <sha>`. Never
  force-push, push to shared branches, or pass `--delete-branch`.

# The reference package is an inventory over engine-owned contracts

The engine owns every contract a package fills: the prompt ids, the base result schemas, the review inputs, the check rule shape, the PR record and the GitHub client. The reference package fills those contracts with defaults and nothing else: prompts, labels, policies and vendored skills. It ships no code, no tier map and no PR body logic in the engine. We chose this so any harness that can read files can run it, and so a user changes method (how to review, which model) without touching what the engine relies on (what a result contains, which commit it binds to).

## Rules

- **Prompt ids.** Required: `review`, `verify` (also triages external comments), `fix`, `resolve-conflict`, `check`, `commit-message`. Optional: `pr-body`, `retro`, `coordinator`. Decision asks and `handoff.md` are engine renders, not package files.
- **Review is a contract.** One review task per round. Its rendered file lists every input: both diffs, acceptance criteria, open findings with supersede chains, resolved one-liners, dropped and deferred findings with reasons, the label taxonomy, the result schema, and the paths of repo guidance files that exist. The coordinator may fan the review out however its harness allows and submits one merged result. The `review` prompt is only the default method.
- **Schemas extend, never loosen.** The engine embeds a base schema per result. A layer may ship `schemas/<result>.json`, validated together with the base (`allOf`). Extra fields are stored as submitted and exposed by `query --json`. Code that acts on them belongs to the extension seam, not to this decision.
- **Labels.** `severity` is required: `blocker | deferrable | nit`. A package policy says only the human may defer or drop a blocker. `axis: spec | standards` and `scope: code | tests | docs | config` are optional. Repo-specific labels such as `area` and `module` belong to the project layer; the fede skill teaches detecting them at setup and in retros.
- **Check rules.** A rule has `kind: command | agent | ci`, an optional `when` path glob that gates dispatch, an optional `instruction` injected into the `check` prompt, and `escalate: auto | always`. A tripwire is a path glob or a `command` rule with `escalate: always`. An `agent` rule is issued as a task, so nothing runs headless (ADR 0001). The coordinator never rewrites a rule. Default tripwires and protected paths: `.fede/**`, `docs/adr/**`, `**/GLOSSARY.md`, `**/AGENTS.md`. Every other rule comes from setup or a retro.
- **The fede skill is embedded** in the binary, outside the layer chain, like a tool's official docs. A package may add a `coordinator` prompt with harness notes; `fede skill` ends with its resolved path.
- **Prompts stay harness-neutral.** They state outcomes and may allow subagents or tools without naming any harness's mechanism. The minimum harness reads files, plus a shell for `check` and `fix`.
- **Tiers are advice.** No tier or thinking ships in prompts or in `fede.yaml`. The fede skill holds the advice table (review flagship/xhigh, verify flagship/medium, fix medium/high, resolve-conflict flagship/high, check medium/medium, pr-body medium/low, commit-message fast/low). Setup writes the user's role map to the project profile; without it, tasks omit `tier` and `thinking`. With fewer than four models, frontier is the highest and fast the lowest, flagship and medium share one model, and a flagship task gains one thinking level.
- **The engine never writes the PR body.** The draft starts with `--fill`. When a layer provides `pr-body`, a writing task runs once before done; `pr_body: off` disables it. The engine's record goes in one PR comment per run, found by a hidden marker and edited in place on every push; a new run's comment marks the previous one superseded.
- **Push gate.** `push_gate: rules | off | always`. `rules` is the default: automatic on the user's own draft, gated when the PR is out of draft or commits touch protected paths, skipped and reported locally when the push is not permitted. Budgets are 3 per category.
- **GitHub through `go-github`.** The token comes from `GH_TOKEN` or `GITHUB_TOKEN`, then `gh auth token` when `gh` is installed. Pushes use the `git` CLI. A preflight before the first outside action checks the token, push permission, `git push --dry-run`, the git user and signing; a failure is a `retry | stop` decision. The record names both identities. Tests use real git with a bare remote and an `httptest` GitHub server, not fakes.
- **Vendored skills pin the latest stable release at vendoring time,** never an arbitrary commit. From mattpocock/skills, pinned by tag (for example `v1.3.1`): `code-review`, `pr`, `retro`, `writing-for-agents`. From cursor/plugins pstack, which has no tags, pinned by the `plugin.json` version plus the commit that set it (for example `0.15.26`): `unslop`, `create-verification-skill` with `references/feature-map-example/`, `maintain-verification-skill`. Each directory carries `LICENSE` and `SOURCE`; `SOURCE` records `version` and `commit`; `pr` keeps `CREDITS.md`.

## Considered options

- **Package-owned result schemas that may change fields:** rejected, the engine derives status, verdicts and budgets from them.
- **Labels as the only extension:** rejected, additive schema extension is safe and cheap.
- **Agent-judged tripwires:** rejected, a tripwire is deterministic; agent rules get the same consequence through `escalate: always`.
- **Tiers in prompt frontmatter:** rejected, tiers are not universal across users' model sets.
- **The fede skill in the layer chain:** rejected, it is fede's documentation, not a package default.
- **`gh` as a required dependency:** rejected, `go-github` plus a token covers every call fede makes.

## Consequences

- Amends ADR 0004: fede talks to GitHub through `go-github`; `gh` is optional.
- Amends the initial design: the PR body is no longer the engine's surface; the record comment carries the claimed decisions (ADR 0010) and deferred findings (ADR 0012).
- `tier` and `thinking` in task items become optional.
- Each release of fede re-vendors the skills at their latest stable release.

# Settings resolve per key, and package settings are namespaced

A setting is a named value that shapes a run. Engine settings (`ci_grace`, `ci_timeout`, `push_gate`, `fixer`, …) are defined by the engine. Package settings are declared by a package, validated by fede and never acted on by the engine. Both resolve per key through the layer chain of ADR 0011, and package settings live under the declaring package's name. We chose per-key resolution because a setting is a single value like `ci_grace` (ADR 0017), not a policy section like a rule list (ADR 0007). Under whole-section replace, setting one key in the project layer would silently reset the package's other keys to their defaults. We chose namespacing because then no new engine key can collide with a package key, so an engine release never breaks a package (ADR 0006).

## Rules

- **Where they are written.** Engine settings are top-level keys in any layer's `fede.yaml`. Package settings go under `settings.<package-name>.<key>`, for example `settings: {fede-pi-herdr-hunk: {walkthrough: always}}`.
- **Resolution.** The chain is a list: repo `.fede/`, project, package, then each package it extends, down to the reference package. For each key, the first layer that sets it wins, and a list or object value replaces the lower one whole. A duplicate key within one file is a parse error.
- **Who may set a key.** Any layer above the declaring package may set its keys. Only the declaring package defines them. The same holds for engine settings: a package may set a default for `fixer`.
- **Declaring.** A package declares its keys in `schemas/settings.json`, a JSON Schema whose `default` keywords supply the defaults. fede validates every layer's values against it when settings load, and an invalid value is a load error. A namespace naming a package that is not in the chain is a warning, and its values are ignored.
- **Read on every call.** Settings resolve on each `next`, so a change takes effect at the next item. Where a setting picks the kind of item, the resolved value is recorded on that item's event in the ledger.
- **Exposure.** `query --json` gains `settings`: every resolved engine and package key with its value and the layer that set it, or `default`. The coordinator reads package settings from there.
- **Ships with slice 2,** alongside `query`. In slice 1, fede-pi-herdr-hunk's `walkthrough` acts as its default, `ask`.

## The `fixer` setting

`fixer: implementer | fresh`, default `fresh`, from slice 7. Before slice 7 every fix behaves as `implementer`.

- **`implementer`.** A fix disposition yields the slice 1 wait `on: "fix"` with `handoff.md`. Whoever implemented the change answers it by committing in the owning worktree (ADR 0014). That may be an agent the coordinator reaches, so the loop can still run unattended. The implementer path does not use the `fix` prompt.
- **`fresh`.** A fix disposition yields a `task` in role `fixer`, rendered from the `fix` prompt, with its submit validated per ADR 0014.
- **The item kind is the signal.** Items gain no `fixer` field. The coordinator acts on the kind it receives.
- **No reachable implementer.** The engine cannot tell who answers a wait, so the fallback is advice in the `coordinator` prompt and the fede skill: hand `handoff.md` to the human. The way to switch a run to `fresh` is a settings change, never a flag on `next`.
- The review-fix budget applies the same way under both values.

## Considered options

- **Whole-section replace for package settings:** rejected, setting one key would reset the others.
- **Flat package keys with a load error on collision:** rejected, a new engine key could break existing packages on upgrade.
- **A fixer task under both values, with a routing field:** rejected, the field is unenforceable and adds a submit step to the implementer path.
- **Default `implementer`:** rejected, it needs a harness that can reach the implementer; `fresh` works with any harness that dispatches a subagent.
- **Settings snapshotted at run start:** rejected, a stuck run could not be switched to `fresh` without stopping it.

## Consequences

- Amends ADR 0011 and ADR 0013: the package layout gains the optional `schemas/settings.json`, and packages may ship `fede.yaml` settings for keys they did not declare.
- Renaming a package orphans users' settings for it; the unknown-namespace warning is the only signal.
- Two packages cannot share a key; each declares its own.

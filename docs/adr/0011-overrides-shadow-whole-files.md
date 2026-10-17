# Overrides shadow whole files and whole skill directories

A package or project replaces one prompt or one skill of what it extends by placing a file or skill directory at the same relative path in a higher layer. There is no override list in `fede.yaml`, no patch format and no include mechanism. We chose plain shadowing because the stable contract is the set of names either way, and a manifest would only add indirection.

## Rules

- **Layout is engine-defined.** A package is `fede.yaml`, `prompts/<prompt-id>.md` and `skills/<name>/`. Prompt ids are engine names, documented with the roles (ADR 0003), and change only additively (ADR 0006). Skill names belong to the package. Packages need not copy the reference package; they ship any subset and inherit the rest.
- **Lookup chain.** Repo `.fede/` in fede's worktree, then `~/.fede/projects/<project>/`, then the package, then each package it extends, down to the reference package. The first layer holding a path wins. All layers share the layout.
- **Units.** A prompt is replaced as a whole file. A skill is replaced as a whole directory, never merged per file: agents resolve a skill's relative files against its own directory, and `SOURCE` and `LICENSE` must describe one version.
- **Prompts name skills.** A prompt lists the skills it uses by name in its frontmatter. fede resolves each name through the chain and puts absolute `SKILL.md` paths in the task, so overriding a skill reaches every prompt that uses it. Dropping a skill means overriding the prompt.
- **The repo layer is read at the head commit.** It is the `.fede/` in fede's own worktree, which sits at the handed-over commit. A change touching `.fede/` trips a default tripwire, so a change cannot quietly rewrite the prompts that review it.
- **Load-time checks.** An unknown prompt id is an error. A skill no prompt references, a relative link in an override `SKILL.md` to a file missing from its directory, and an override whose `SKILL.md` hash differs from `SOURCE` while `local_changes` is empty are warnings. Whoever edits an override writes `local_changes`; fede only checks it.
- **Drift.** Each override records `based_on`, the sha256 of the base file it replaced: frontmatter for prompts, `SOURCE` for skills. When the base no longer matches, main commands print a notice naming the base's source and path. A missing `based_on` gives a "base unknown" notice. fede keeps no state for this. The resolved layer map appears in `query --json`.

## Considered options

- **Per-file merge of skill directories** with fede materializing a combined copy: rejected, it ships untested combinations and breaks provenance.
- **Skills as a union namespace** where collisions are errors: rejected, users would override every prompt that names the skill.
- **Reading the repo layer from the default branch:** rejected, it blocks iterating on prompts in a branch; the tripwire covers the self-review risk.

## Consequences

- Only local bases are compared. Detecting new commits in remote package sources or in vendored skills' upstreams belongs to the later versioned-sources work, which feeds the same `based_on` check.
- The fede skill teaches agents to create overrides with `based_on`, merge base changes on a drift notice, prefer a prompt override for small tweaks to a vendored skill, and run a retro on the ledger that proposes changes to a chosen layer.

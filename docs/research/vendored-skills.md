# Vendored skills: provenance and licenses

Ticket: AntiD2ta/fede#4. Upstream state observed 2026-10-17.

## Summary

| Skill | Upstream (path) | Pin | License | Transitive deps | Verdict |
|---|---|---|---|---|---|
| `pr` | mattpocock/skills `skills/engineering/pr` | Tag `v1.3.1` (24fe0ef, 2026-10-04). Path unchanged since e484a80 (2026-09-24) | MIT, Matt Pocock 2026 | `show-me` (content copied, not a dependency). `GLOSSARY.md` (consumer repo) | Vendor upstream, with LICENSE and CREDITS.md |
| `unslop` | cursor/plugins `pstack/skills/unslop` | `main` 101b274 (2026-10-17). Path last changed bc697d0 (2026-10-12) | MIT, Lauren Tan 2026 | None | Vendor upstream, not the local copy (Q1) |
| `code-review` | mattpocock/skills `skills/engineering/code-review` | `main` 49dd158 (2026-10-09). Path last changed 3da8c01 (2026-10-07) | MIT, Matt Pocock 2026 | `docs/agents/issue-tracker.md` (consumer repo, made by `setup-matt-pocock-skills`). Host sub-agents | Vendor SKILL.md. Do not vendor setup skill (Q2) |
| `create-verification-skill` | cursor/plugins `pstack/skills/create-verification-skill` | `main` 101b274 (2026-10-17). Path last changed c49b37a (2026-10-12) | MIT, Lauren Tan 2026 | `references/feature-map-example/` (required). `/maintain-verification-skill` (Q3). `writing-for-agents` (soft) | Vendor with feature-map-example |

## Attribution rule (all four)

All four are MIT. Each copy or substantial portion must keep the copyright line and the permission notice. Vendor each skill's LICENSE next to its SKILL.md. The root Apache-2.0 license is unchanged. Under my reading, Apache-2.0 adds no notice duty for MIT material, so confirm this with the fede maintainers.

## Per-skill findings

### pr

- **F1 Upstream.** https://github.com/mattpocock/skills/tree/main/skills/engineering/pr. Path files: `CREDITS.md`, `SKILL.md`, `agents/`. There is no skill-level LICENSE. The root LICENSE (`Copyright (c) 2026 Matt Pocock`, MIT) applies.
- **F2 Pin.** Tag `v1.3.1` (commit 24fe0ef). The path has not changed since e484a80 (2026-09-24), so the tag copy matches `main`.
- **F3 Show-me.** `CREDITS.md` says `pr` does not depend on `show-me` and copies its Summary section. The copied text came from https://github.com/humanlayer/skills/blob/main/plugins/show-me/skills/show-me/SKILL.md. Its license is MIT, `Copyright (c) 2026 HumanLayer` (verified). Ship that notice with the copied text. Do not vendor `show-me` itself.
- **F4 Consumer file.** The body says to use the domain language from `GLOSSARY.md`. This is a repo-supplied file and is not vendored.
- **F5 Differences.** Frontmatter only, as far as the read showed. Local adds `license: MIT` and `metadata.source/author/url`, and turns `credits` into a string. Upstream nests `credits` as an object. The body reads the same. I did not byte-compare.

### unslop

- **F6 Origin.** The local copy has no source metadata. Its origin is cursor/plugins `pstack/skills/unslop`, added in commit 24bd6eb (2026-05-23, "Add pstack plugin"). Third-party mirrors labelled "unslop Skill by cursor" carry the same wording as the local copy.
- **F7 License.** `pstack/LICENSE` (MIT, `Copyright (c) 2026 Lauren Tan`). `pstack/.cursor-plugin/plugin.json` also states `"license": "MIT"`. The clone has no root LICENSE file.
- **F8 Pin.** `main` 101b274 (2026-10-17). Path last changed bc697d0 (2026-10-12, #572).
- **F9 Differences.** The local copy differs from upstream in three places.
  1. Description: local keeps "Must always apply." Upstream removed it in #572 because the skill sets `disable-model-invocation: true`.
  2. Frontmatter: local lacks `disable-model-invocation: true`. Upstream has it, set in #300 (2026-09-01).
  3. Body: local has an "Adding soul" section and a rule list numbered from 1. Upstream has no "Adding soul" section and starts its rule list at 3.
- **F10 Dependencies.** None in `SKILL.md`.

### code-review

- **F11 Upstream.** https://github.com/mattpocock/skills/tree/main/skills/engineering/code-review. Files: `SKILL.md` (blob 373a4f2), `agents/openai.yaml`.
- **F12 Pin.** `main` 49dd158 (2026-10-09). Path last changed 3da8c01 (2026-10-07, #1208). Tag `v1.3.1` (2026-10-04) predates #1208, so the tag copy differs. I did not check how.
- **F13 Tracker file.** The skill says the issue tracker "should have been provided", and to run `/setup-matt-pocock-skills` if not. That skill lives at `skills/engineering/setup-matt-pocock-skills` (MIT). Upstream ADR 0001 (`.agents/adr/0001-explicit-setup-pointer-only-for-hard-dependencies.md`) treats a skill with this one-liner as a hard dependency. Vendoring the setup skill is not required. fede must supply `docs/agents/issue-tracker.md` in the consumer repo, or the skill fails.
- **F14 Other deps.** Sub-agents are a host capability, not a file. The smell baseline is inline text citing Fowler's *Refactoring*, ch. 3. `CODING_STANDARDS.md` and `CONTRIBUTING.md` are consumer files.
- **F15 Differences.** Frontmatter adds `license: MIT` and `metadata`. The body names `docs/agents/issue-tracker.md` explicitly in two places. Upstream says only "the tracker doc".

### create-verification-skill

- **F16 Upstream.** https://github.com/cursor/plugins/tree/main/pstack/skills/create-verification-skill. Pin 101b274. Path last changed c49b37a (2026-10-12, #573). The local copy has its own LICENSE with the same MIT text as `pstack/LICENSE`.
- **F17 Required file.** `references/feature-map-example/` (`README.md`, `create-note.md`, `search.md`) is linked from step 3. Vendor it.
- **F18 Maintenance pointer.** Step 6 offers `/maintain-verification-skill` (`pstack/skills/maintain-verification-skill`, same MIT license). It is not present locally. See Q3.
- **F19 Soft dependency.** The local copy says to load `writing-for-agents` if installed. Upstream home is `skills/productivity/writing-for-agents` in mattpocock/skills (MIT). A local copy exists with a LICENSE. The skill works without it, so do not vendor it.
- **F20 Differences.** Local, against upstream:
  - Output path is `.agents/skills/` everywhere. Upstream uses `.cursor/skills/`.
  - Local adds a publication-policy paragraph and a `.claude/skills` symlink offer in step 2.
  - Local adds the `writing-for-agents` line.
  - Local adds a new step 5 on `AGENTS.md`/`CLAUDE.md`, so local has 6 steps to upstream's 5.
  - Local adds `license` and `metadata` to the frontmatter.
  - Local has a local-only tool mention in the Drive step.
  - Step 4 and the feature-map text are not byte-compared.

## Vendoring procedure and SOURCE format

Vendor the upstream file at the pinned commit (`git show <commit>:<path>/SKILL.md`), not the local copy. Record each local edit in `local_changes`.

```yaml
# SOURCE
skill: create-verification-skill
upstream: https://github.com/cursor/plugins
path: pstack/skills/create-verification-skill
commit: 101b274dd991f694e1f7024be2157e5f3adf07de   # pinned commit
path_commit: c49b37a2c0e7bede60a71ad7322e3f12c337a3da   # last commit touching path
retrieved: 2026-10-17
license: MIT
copyright: Copyright (c) 2026 Lauren Tan
license_file: LICENSE
sha256_skill_md: <computed at vendoring>
local_changes: none
```

## Unknowns

- **Q1 Unslop snapshot.** The exact upstream commit behind the local `unslop` text is unknown. The local text is closest to the upstream file before #572. Bound it with `git log -p -- pstack/skills/unslop/SKILL.md` in the clone.
- **Q2 Tracker setup.** Should fede require consumers to provide `docs/agents/issue-tracker.md`, or vendor the setup skill? This is a fede maintainer decision. I recommend requiring the file.
- **Q3 Maintenance skill.** Vendor `/maintain-verification-skill`, or remove the step 6 pointer? This is a fede maintainer decision.
- **Q4 Tag or commit.** Pin the `v1.3.1` tag for `pr`. For `code-review`, the tag is stale, so pin commit 49dd158.
- **Q5 Unverified.** `writing-for-agents` license text was not read. The `show-me` text was not compared to the `pr` Summary. The local `feature-map-example` was not compared to upstream.

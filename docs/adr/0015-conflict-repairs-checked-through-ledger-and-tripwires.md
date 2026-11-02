# Conflict repairs are checked through the ledger and tripwires

When the branch conflicts with its base, a conflict resolver brings the base in and writes a conflict report. The engine then checks the result with git and the machinery it already has: tripwires and the ledger's `supersedes` links. It adds no new escalation trigger and asks the resolver to declare nothing about recorded decisions. A recorded decision is a human disposition in the run's ledger, or a decision committed in the target repo's protected paths. We chose this because the re-reviewer is independent of the resolver, and a tripwire is deterministic, while a resolver's own claim about what it kept would be one more unchecked agent statement.

## Rules

- **Detection.** fede fetches the base and runs `git merge-tree --write-tree <head> <base>` locally. GitHub's `mergeable` state is not used. The conflicts step runs first in every round, before review, and the repair re-enters review and checks (P5).
- **Strategy.** `conflict_strategy: rebase | merge` is a package policy; `rebase` is the default. A rebase push is a force push with a lease anchored to the last remote SHA fede saw (P4), under the usual push gate.
- **The resolver works end to end** in the owning worktree (ADR 0014): it merges or rebases, resolves and commits. After submit the engine checks a clean worktree with no merge or rebase in progress, the named base SHA is an ancestor of the new head, and `git merge-tree` against it is clean. For `merge`, the old head is an ancestor of the new head. For `rebase`, every old commit has a counterpart by subject, in order, or is listed in the report's `dropped`; an unlisted missing commit rejects the submit.
- **Conflict report.** `paths[]` with `path`, optional `commit` (rebase stops), `resolution: head | base | combined | rewritten`, `kept_from_head`, `kept_from_base` and `rationale`; `questions[]` in the finding shape; `dropped[]` with `commit` and `reason`; an optional free-text `ran`. Every path from the initial `merge-tree` needs an entry. Revisions (head, base, merge base, result) are recorded by the engine, not submitted (ADR 0006). Verification is the re-review and checks that follow, not the report.
- **Questions** become `source=conflict` findings that the verifier triages; it may escalate them through trigger 2.
- **Tripwires over conflicted paths.** Tripwire globs are evaluated against the conflicted path set, not only the branch diff, because taking the base side removes a path from the diff. Each match becomes a `source=conflict` finding carrying that path's report entry, escalated after the resolver acts through trigger 1. Drop accepts the resolution; fix sends it back with the human's note.
- **Undoing a ledger decision.** After a conflict repair, the re-review input lists in full every resolved, dropped or deferred finding located in a conflicted path, and asks the re-reviewer to raise a superseding finding if the resolution brought it back. ADR 0012 classifies the link: a returned finding (trigger 4) or a superseded drop or defer (trigger 3).
- **Budgets stay per category.** A round with a conflict repair costs one conflict-repair attempt; a fix in the same round also costs one review-fix attempt.
- **Where the report goes.** The full report is rendered into the next re-review input and exposed in `query --json`; the record comment gets one line per conflicted path with its resolution.

## Considered options

- **Merge only:** rejected, many repos keep linear history.
- **The engine drives git and the resolver only edits files:** rejected, the resolver handles a rebase better end to end; the engine re-checks the outcome with the same `merge-tree` it detected with.
- **GitHub's mergeable state for detection:** rejected, it is asynchronous, sometimes unknown, and lists no paths.
- **The resolver declares `kept | changed` per recorded finding:** rejected, a second agent claim adds a field and a trigger without adding determinism.
- **A new trigger for resolver questions:** rejected, many are answerable from the code, and the verifier exists by then.
- **One shared repair budget per round:** rejected, a fast-moving base would eat the fix budget.

## Consequences

- Amends ADR 0014: under `rebase`, the resolver's submit uses the checks above instead of requiring descent from the issued head. Under `merge`, ADR 0014's rule holds.
- Tripwire rules apply to two path sets: the branch diff and, in a conflict round, the conflicted paths.

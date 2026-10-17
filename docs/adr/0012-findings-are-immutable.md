# Findings are immutable; status is derived

A finding, once raised, never changes: its id, claim, location, source, labels and the commit it was raised at are fixed. Dispositions and finding verdicts are events appended to the run's log, and a finding's status is computed from them. When a reviewer sees a known problem again with new context, it raises a new finding that `supersedes` the old one. The engine reads the old finding's status to classify the link: superseding an open finding is a refinement, which inherits the disposition; superseding a resolved finding is a returned finding, which escalates early under the repair budget; superseding a dropped or deferred finding is always a decision for the human. We rejected reopening the same finding because that loses the reviewer's updated description, and we rejected asking reviewers to label regressions because the engine can tell from state they cannot see.

## Consequences

- There is no `fixed` status. A fix is only resolved by a finding verdict or a passing check at a newer commit, or at the same commit when the check failed for environment or flaky causes.
- `submit` rejects a review that leaves any open finding without a finding verdict.

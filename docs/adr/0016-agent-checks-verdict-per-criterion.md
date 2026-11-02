# Agent checks return one verdict per acceptance criterion

An agent check returns one criterion verdict for each acceptance criterion in the run's current snapshot, and the engine derives everything else: the check verdict, the findings and their resolution. Findings key on the criterion, not on the rule that judged it, so a criterion failing under two rules or across commits is one problem with one history. Criteria are copied from a source the human controls and never guessed, and only the human may change them mid-run. We chose this so the engine acts on fixed, typed fields (ADR 0013) and an agent cannot make a failing criterion disappear by rewording or re-reading it.

## Rules

- **Criterion verdict.** `{criterion, verdict: pass | fail | blocked | n/a, notes, evidence: [{path, caption}]}`. `notes` is required on `fail`, `blocked` and `n/a`, and optional on `pass`. `blocked` means the agent could not exercise the criterion (the app would not start, seed data was missing). It is not a claim that the change is wrong. A pass with notes is shown as such, but it is not a separate value, and its notes never raise a finding.
- **Coverage.** The engine numbers criteria `AC1`, `AC2`, … in the rendered `check` task. Every agent rule's task returns exactly one verdict per id, and `submit` rejects gaps and unknown ids. A rule's check verdict is fail, then blocked, then pass, ignoring `n/a`.
- **Several agent rules.** Every agent rule gets every criterion. A rule runs at head when its `when` glob matches the whole change (base to head). Once every agent rule dispatched at head has submitted, the engine combines each criterion's verdicts: `fail` if any rule failed it, else `pass` if any passed it, else `blocked` if any was blocked, else uncovered (every rule said `n/a`).
- **Evidence is optional.** Each path must lie inside the evidence directory fede names for the task. At submit the engine checks that each file exists and records its sha256. The record marks a pass with no evidence as unevidenced. Some criteria have no useful artifact, and the human sees which ones at the ready packet.
- **Bound to the issuing head.** The result carries no commit. It binds to the head the task was issued at (ADR 0005), and a submit after head moved is stale.
- **Findings.** A combined `fail` raises one finding per criterion. A `blocked` criterion still blocked after one re-run of the blocked rules at the same commit, or an uncovered one, raises one "not verified" finding. Both have `source=check`, the criterion as title, the notes as claim, no `path:line`, and labels `severity: blocker, axis: spec`. Until the verifier exists, the human disposes them (escalation trigger 6, ADR 0012).
- **Resolution.** A combined pass at a newer commit resolves the finding, or at the same commit for a "not verified" finding, since its cause is the environment (ADR 0012). A repeat fail at a newer commit keeps the open finding and costs one check-fix attempt. A fail after resolution makes the engine raise a new finding that supersedes the resolved one, which is a returned finding.

## Acceptance criteria intake

- **One coordinator task.** The first time an agent rule would dispatch in a run, `next` issues a `criteria` task for the coordinator. The task says where to look, in order: the linked ticket in the user's tracker, then the PR body's `## Acceptance criteria` section if a PR exists, then the human. Criteria are copied from the source, never inferred from the diff or the code. Anything unclear goes to the human. The result is `{criteria: [text], source: <url> | "human"}`, and `source: "human"` requires `--by human` (ADR 0010).
- **Snapshots.** Each accepted result is a snapshot event in the run's log. Every agent check verifies against the current snapshot, and earlier snapshots stay readable through `query`.
- **Re-snapshot.** `fede next --criteria` issues a fresh `criteria` task when the coordinator learns the source changed, and a pending agent `check` task is voided. The engine diffs by exact text. An unchanged criterion keeps its id, a new one gets the next unused id, and a missing one is retired. An edited criterion is retire-plus-add, and ids are never reused. An identical snapshot is a no-op.
- **Only the human changes the criteria.** A snapshot that changes the list requires `--by human`. It voids every agent verdict at head. The engine appends a `drop` disposition to each open finding on a retired criterion, with the reason "criterion retired in snapshot N", attributed to that claim.
- **Explicitly none.** An empty list is valid only with `none_quote`, the source's own words saying no criteria apply, or with `--by human`. Agent checks are then skipped, and the record says why.

## Considered options

- **A separate `pass-with-notes` value:** rejected, a pass with non-empty notes says the same thing in one shape.
- **One agent rule per project:** rejected, projects verify different surfaces (a TUI, an API). Rules can't know ahead of time which criteria they cover, so every rule judges every criterion and may answer `n/a`.
- **Required evidence:** rejected, some criteria have no artifact that represents them better than the notes. Unevidenced passes are made visible instead.
- **Findings keyed on rule and criterion:** rejected, two rules failing one criterion would be two findings for one problem.
- **Fuzzy matching of edited criteria:** rejected, it is a judgment, and the engine does not make judgments.
- **Engine-side lookup of criteria on GitHub:** deferred, the PR usually does not exist yet when checks first run, and the coordinator task covers every tracker.

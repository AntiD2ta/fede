# Implementer evidence is trusted only through the verifier's acceptance

By default fede re-runs every agent check itself. A project may opt in to trusting the implementer's evidence instead, but only at the handed-over head and only for the criteria an independent verifier accepts. The verifier reads the implementer's claims and artifacts and accepts or rejects each one. It never fails a criterion. Every rejected, missing or non-passing claim goes to a live agent check limited to those criteria. A criterion therefore fails only when an agent exercised the change at that head, and trust only removes the duplicate run when the implementer already did the work. This is the explicit, scoped exception that ADR 0005 allows. The engine default stays strict.

## Rules

- **Setting.** `implementer_evidence: rerun | trust` is an engine setting (ADR 0019), default `rerun`. It covers every agent rule at once. The value that applied is recorded on the `evidence` task's event.
- **Scope.** Trust applies only at the run's first head, the handed-over commit, and only under the criteria snapshot the implementer's claims were mapped to. Any later head (a handoff fix, a fixer commit, a rebase or a conflict repair), a snapshot that changes the list, and the re-run of a blocked criterion all run live.
- **Intake.** When trust applies, `next` issues an `evidence` task to the coordinator after the criteria snapshot (ADR 0016). The coordinator asks the implementer for its artifacts, copies them into the evidence directory fede names for the task, and maps them to `AC1`, `AC2`, …. The result is `{commit, claims: [{criterion, verdict: pass | fail | blocked | n/a, notes, evidence: [{path, caption}]}]}`, or `{none: true}`. The engine refuses a `commit` other than head, checks that each file exists and records its sha256. Where the files came from does not matter once they are copied. The submit is a claim (ADR 0010).
- **Which claims reach the verifier.** A claimed `pass` or `n/a`. A `pass` may rest on notes alone, for example when the diff itself proves a documentation criterion, and the record marks it unevidenced. A claimed `fail` or `blocked`, and a criterion with no claim, go straight to the live run.
- **Judging.** The verifier role judges the claims through a second section of the `verify` prompt. The task is read-only and has no shell. Its inputs are the criteria with ids, the diff base to head, each claim with its artifact paths and checksums, and the agent rules' `instruction`s. Each claim gets `accept` or `reject` with notes. The result also carries `suggested_checks: [{criterion, check, why, needed}]`, where `criterion` is optional.
- **Suggested checks.** A needed check means the verifier cannot accept that criterion without running something, so it must reject it. `submit` refuses an accept whose criterion has a needed check. The needed check is rendered as a hint for that criterion in the live `check` task. A check that is useful but not needed is recorded in the ledger and listed in the record, and a retro or setup may turn it into a `command` rule. The coordinator never runs a suggested check itself.
- **Trusted verdicts.** An accepted claim becomes that criterion's verdict at head with the implementer's evidence, shown as trusted, never as checked.
- **Live run.** If any criterion is left over, every agent rule whose `when` glob matches runs with only those criteria. Coverage and the combine rule of ADR 0016 apply to that subset. A reject raises no finding, and the record lists each one with its notes. A live run is not a repair, so it costs no budget.
- **Independence is advice.** The fede skill advises that the verifier be a fresh agent, never the implementer's session. fede cannot enforce it, because who ran a task is a claim (ADR 0010).
- **Command and CI checks are never trusted.** fede runs `command` rules itself at every head (ADR 0018) and watches CI (ADR 0017). Tests, linters and mutation testing belong there.

## Considered options

- **The verifier returns full criterion verdicts:** rejected. A reading of someone else's artifacts could fail the change with no agent having exercised it.
- **Any reject re-runs every criterion:** rejected. It repeats the work the accepted claims already cover.
- **Per-rule trust (`trust: implementer` on an agent rule):** rejected. Implementer evidence is keyed by criterion, not by rule, so it is unclear which rules an accepted claim replaces.
- **Every pass needs an artifact:** rejected. Small changes such as a typo or a docs edit often have no artifact better than the diff, and the gate would force a live run on them.
- **The implementer submits evidence before `next`:** rejected. No run or criteria ids exist yet.
- **Evidence committed to the branch:** rejected. It pollutes the change under review.
- **A shell for the verifier:** rejected. Running tests or building a client is verification, not judgment. It would make the verifier a second, unbounded check. Execution belongs to command rules or the live run.
- **A separate `judge-evidence` prompt id:** rejected. `verify` already holds more than one job (ADR 0013).
- **Trust at any head with matching evidence:** rejected. Initial design grilling settled that live checks re-run after any repair.

## Consequences

- Amends ADR 0016: a criterion's verdict at head may be trusted, and a live run may cover a subset of the criteria.
- Amends ADR 0013: the coordinator gains the `evidence` task, and the `verify` prompt gains a section for judging evidence.
- Adds a `suggested_checks` input to retros and setup.
- Ships with slice 6, after the verifier (slice 5).

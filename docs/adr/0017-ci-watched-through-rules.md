# CI is watched through rules, and the default rule watches every check

fede watches only the CI checks a `ci` rule names. The reference package ships one rule, `{kind: ci, name: "*"}`, which watches every check reported on the head, and a project narrows it with an override. We chose this because some workflows are advisory, known to be flaky or expected to fail, and which ones matter depends on the project and the user, not on GitHub's settings. A failed CI check is a finding like any other; whether it was caused by the change is a `cause` the human gives until the verifier exists, and a deterministic stale-base rule runs before any failure may be called flaky.

## Rules

- **Rule shape.** A `ci` rule requires `name`, a glob matched against check run names and commit status contexts. Every matched check must pass. `when`, `instruction` and `escalate` keep their ADR 0013 meaning; `escalate: always` sends every failure of that check to the human, after the verifier exists too.
- **Skipping.** With no `ci` rule applying, the CI step is skipped. A rule that applies but matches no check within `ci_grace` of fede's push of that head is recorded as "no matching check" and skipped. Failing checks no rule covers raise no finding; the record lists them one line each.
- **Settings.** `ci_grace` (default `3m`) and `ci_timeout` (default `30m`) are engine-owned: the engine holds the key, the default and the validation, and any layer may set the value. A value of zero or below is rejected when settings load.
- **Timeout.** When watched checks are still pending `ci_timeout` after fede's push of the head, one decision covers them all: `keep-waiting | skip | stop`. `skip` applies to that head only and never to a failed check; it is a claim (ADR 0010) listed in the record, and the ready packet's "CI green" excludes the skipped checks.
- **Waits.** A wait on pending checks is `on: "ci"` with `after`, a hint in seconds (30 for CI). `update-base` yields `on: "base"`, cleared when the target tip recorded at that decision is an ancestor of head. `next --wait` polls every 30 s and returns after at most 90 s by default.
- **Cause.** Until slice 5, the human answers a CI finding with a disposition (fix, which implies cause `change`, drop or defer) or a cause, `flaky` or `environment`. The engine records a `cause` event, the same event the verifier emits later. A cause is not a disposition; the set stays fix, drop, defer (ADR 0012).
- **Stale base.** fede records the target tip at each push. The base is stale for a CI failure when the target tip has moved since the push that triggered the failing run. Being behind the target, or mergeable with it, plays no part. While the base is stale, `flaky` and `environment` are refused, and the decision offers fix, drop, defer or `update-base`.
- **Fails on base.** When the same check also fails at the target tip, the finding is marked "fails on base" and may be deferred. The ready packet allows red CI only for those deferred findings, and the record lists them.
- **One retry.** A `flaky` or `environment` cause buys one retry per check per head, outside the check-fix budget. fede re-runs Actions jobs itself; for other checks it asks the human to re-trigger. A pass at the same commit resolves the finding (ADR 0012). A second failure escalates: after `flaky` with only fix, drop and defer offered; after `environment` as a `retry | stop` blocker decision, where human retries are uncapped.

## Considered options

- **Watch every check on the head with no rule:** rejected, advisory and expected-to-fail workflows would block every run, with no way to configure it.
- **Watch GitHub's required checks only:** rejected, it ties fede to repository settings the user may not control and ignores useful non-required checks.
- **Stale means behind the target:** rejected, a branch that is behind but merges cleanly is normal.
- **Skip a rule as soon as it matches nothing:** rejected, checks appear some time after a push, so done could come before CI starts.
- **`retry` as a fourth disposition:** rejected, flakiness is a cause, which the verifier will report anyway.

## Consequences

- Amends ADR 0013: besides the default tripwires, the reference package ships the default `ci` rule, and a `ci` rule gains the required `name` field.
- Adds the `ci` and `base` wait blockers and the `after` field to wait items (ADR 0006, additive).
- Slice 5 changes only who emits the `cause` event.

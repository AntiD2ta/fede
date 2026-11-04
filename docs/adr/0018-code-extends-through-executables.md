# Code extends fede through executables, not an embedded runtime

Packages and users add behavior in code only through the exec seam: fede runs the program a `command` check rule names, reads its exit code, and reads findings from a JSON file the program may write. Everything else outside fede reads state through `query --json`. Check rules are the only code hook. Renderers and handlers for extended schema fields (ADR 0013), such as Hunk notes or a tracker issue per deferred finding, run in the harness, driven by a package's `coordinator` prompt. We chose this because a separate process works in any language and keeps a package a directory that stock fede can load (ADR 0007). Pi takes the other path, with in-process TypeScript and many lifecycle hooks, because its agent loop is the product. fede's step order is fixed (ADR 0003), so most such hooks would have nothing to attach to.

## Rules

- **fede runs `command` rules itself,** synchronously inside `next`, in its own worktree at the reviewed commit (ADR 0014). Running a process is not spawning an agent, so ADR 0001 holds. A rule gains `run`, a string fede passes to `sh -c`, and `timeout`, which defaults to 10 minutes and may be set in any layer.
- **One `next` runs every pending rule in order** and journals each verdict as its rule ends. Before it starts, `next` returns `{kind: "wait", on: "check", rules, timeout}`, where `timeout` is the sum of the pending rules' timeouts. That sum bounds the call that runs them, so the coordinator can keep it alive that long.
- **Polling.** A `next` that finds the run locked during the checks step returns at once with a progress wait (`done`, `running`, `pending`, and `after: 10`, a hint in seconds as for CI waits in ADR 0017) instead of blocking. `next --wait` polls the same way and returns after at most 90 s. When the harness kills the running call, the OS releases its lock, and the next call re-runs the interrupted rule and the pending ones. A check has no outside effect, so a killed check re-runs instead of failing closed (ADR 0002). A rule killed twice without a verdict at the same head gets the timeout decision below instead of a third run, and the decision names the harness's shell limit as the likely cause.
- **No fail-fast.** A failed or timed-out rule does not stop the sequence. Failures become findings for one round, which costs one check-fix attempt.
- **Timeouts are decisions.** fede kills a rule that reaches its timeout. All timed-out rules in a sequence share one decision, answered with `--by human` (ADR 0010): `{"choice": "retry", "timeout": "<duration>"}` or `{"choice": "stop"}`. The new timeout holds for the rest of the run and is recorded in the ledger.
- **Findings file.** fede sets `FEDE_FINDINGS` to a path. The program may write `{"findings": [{title, path, line, body, labels, key}]}` there, validated against an engine base schema that layers may extend (ADR 0013). A non-zero exit fails the check. With no file, fede raises one finding from the rule name and the tail of the output. Exit 0 passes. Findings written on exit 0 become the notes of a pass and never raise a finding (ADR 0016).
- **Finding keys.** `key` is optional and chosen by the program. At a newer commit, a finding whose key appears again stays open, one whose key is missing is resolved, and a new key raises a new finding. Without keys, a rule's findings resolve together when the rule passes (ADR 0012).
- **Environment.** fede sets `FEDE_BASE` (the base SHA fede reviews against, for diff-scoped checks), `FEDE_FINDINGS`, and `FEDE_LAYER` (the absolute directory of the layer that defined the rule). The head is the worktree's `HEAD`. stdout and stderr go to a file in the run directory as the check's evidence, bound to the commit (ADR 0005).
- **Packages may ship scripts** that rules call through `$FEDE_LAYER`, for example `run: $FEDE_LAYER/bin/check-api`. The reference package ships none (ADR 0013).
- **Trust.** Commands run with the user's permissions. fede adds no gate beyond the `.fede/**` tripwire, because the branch's own tests already run code the agent wrote and the implementing agent already had a shell. The docs say so plainly, as Pi's do for extensions.
- **Go builds are an unpromised escape hatch.** The library stays public (ADR 0008), but in v0.x its exported API may change in any release. It has no registration API and is not documented as an extension seam. A custom binary can't be combined with another custom binary, while packages combine through `extends`.
- **The fede skill teaches the mechanics per harness:** how to keep the running `next` alive (on Claude Code a foreground call that reaches its limit, 2 minutes by default and at most 10, moves to the background and keeps running for up to 10 more minutes; opencode needs a background call; Pi takes a long `timeout`), polling `next` for progress, and handling the timeout decision.

## Considered options

- **An embedded runtime** (TypeScript through esbuild and goja, or WebAssembly through wazero): rejected. fede would ship a runtime plus an in-process API it must version additively (ADR 0006), and in-process event hooks conflict with the fixed step order. Few package authors would write WebAssembly checks.
- **Custom Go builds as the main seam:** rejected. A package that needs its own binary is no longer a directory any stock fede loads.
- **Event hooks** (`on_push`, `on_done`, per-field handlers): deferred until a package needs one. The coordinator already acts on `query --json`.
- **The coordinator runs commands as tasks:** rejected. The exit code would become a claim, and an agent would sit inside a deterministic step (P1).
- **A detached child process per check:** deferred. It survives any harness shell limit but needs pid tracking, orphan cleanup and `fede stop` handling, which ADR 0002 avoided.
- **Parallel rules in one call:** deferred. Rules share one worktree and can contend for ports, databases and build directories.
- **One rule per `next` call:** rejected. The coordinator would make one call per rule for no gain, since per-rule journaling already limits a re-run to the interrupted rule.
- **A `--timeout` flag on `next`:** rejected. The coordinator never rewrites a rule (ADR 0013), and a flag would let it lengthen a check silently.

## Consequences

- Amends ADR 0013: a `command` rule gains `run` and `timeout`, and may return findings.
- Extends ADR 0012: a check finding with a `key` resolves when its key disappears at a newer commit, not only when the whole check passes.
- Adds the `check` wait blocker, which uses the `after` field from ADR 0017 (ADR 0006, additive).
- Detached runs and parallel rules can land later without changing the item shape.

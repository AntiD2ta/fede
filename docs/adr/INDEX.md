# ADR index

Every architecture decision record in `docs/adr/`, with its topic. Update this table whenever an ADR is added, superseded or retitled.

| ADR | Title | Topic or concern | Status |
|---|---|---|---|
| [0001](0001-harness-runs-the-agents.md) | The harness runs the agents; fede never spawns them | Execution model | accepted |
| [0002](0002-no-daemon-state-on-disk.md) | No daemon: a CLI with durable state on disk | State, resume, idempotency | accepted |
| [0003](0003-fixed-step-order.md) | The step order is engine code; packages configure it | Pipeline, extensibility | accepted |
| [0004](0004-github-only-platform.md) | GitHub is the only platform fede talks to | Integrations, scope | accepted |
| [0005](0005-results-bound-to-a-commit.md) | Every result is bound to the commit it ran on | Verification, evidence | accepted |
| [0006](0006-minimal-additive-protocol.md) | The item protocol starts minimal and only grows additively | Coordinator contract | accepted |
| [0007](0007-packages-are-directories.md) | Packages are directories that extend by file | Packages, customization | accepted |
| [0008](0008-go-library-first.md) | fede is a Go library first, with a thin CLI | Architecture, language | accepted |
| [0009](0009-fede-never-merges.md) | fede never merges | Scope, human control | accepted |
| [0010](0010-submitter-is-a-claim.md) | Who submitted a result is a recorded claim, not authentication | Coordinator contract, human control | accepted |
| [0011](0011-overrides-shadow-whole-files.md) | Overrides shadow whole files and whole skill directories | Packages, customization | accepted |

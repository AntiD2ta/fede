# No daemon: a CLI with durable state on disk

fede is a plain CLI. Run state lives under `~/.fede` (overridable with `FEDE_HOME`): an append-only event log per run plus artifacts referenced by path. Progress happens only while a coordinator calls fede, and any new session resumes by calling `fede next`. We rejected a daemon (the no-mistakes model) because the harness already provides the long-lived process, and a daemon adds service installation, IPC and recovery code.

## Consequences

- Every deterministic step journals `started` and `done`. A step found started but not done fails closed into a decision item; fede does not auto-recover.
- Outside actions check real state before acting (existing PR for the branch, reply markers, remote SHA), and `submit` is idempotent per item.
- Concurrent calls on one run are serialized with a file lock on the run directory.

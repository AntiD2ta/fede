# The harness runs the agents; fede never spawns them

fede does not launch agents, headless or otherwise. A coordinator agent inside the user's harness loops: `fede next` returns one item, the coordinator runs any task through its own subagent mechanism (Claude Code subagents, Codex subagents, Pi through Herdr or tmux, opencode), and returns the result with `fede submit`. We chose this over headless runners because it keeps agent work visible and steerable, reuses the user's skills, models and subscription, and removes one adapter per harness. Headless runners remain possible as a user-built extension.

## Consequences

- fede still decides every next step (the coordinator only dispatches), so the pipeline stays deterministic.
- Long waits are expressed as `wait` items; the coordinator decides how to sleep, and fede holds no agent context.

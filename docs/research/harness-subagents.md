# Harness subagent capabilities for a fede coordinator

Direct child agents: none.

Scope: ticket AntiD2ta/fede#3. Every page below is undated, so versions come from the sources where given. Pi claims were checked against a local checkout of the Pi repo at commit `f9b510e2` (package 1.1.0). Links point to the upstream paths. Caching and pricing from the prior survey are out of scope and not repeated here.

## Summary matrix

| Harness | (a) cwd | (b) model | (c) effort / thinking | (d) read-only tools | (e) skill by path | Parallel | Visible to user |
|---|---|---|---|---|---|---|---|
| Claude Code | partial | yes | partial | yes | partial | yes | yes (interactive) |
| Codex CLI | partial | yes | yes | partial | partial | yes | yes (TUI); unknown in `exec` |
| opencode | unknown | partial | yes (provider-dependent) | yes | partial | yes | yes |
| Pi | yes | yes | yes | yes | yes | yes (separate processes) | partial (needs a pane) |

## Claude Code

- **Spawn and config.** The `Agent` tool uses `model`, `effort`, `tools`, `disallowedTools`, `skills`, and `isolation` from agent frontmatter or the `--agents` JSON ([sub-agents](https://code.claude.com/docs/en/sub-agents), [CLI reference](https://code.claude.com/docs/en/cli-reference)).
- **(a) partial.** "A subagent starts in the main conversation's current working directory." There is no per-spawn cwd. `isolation: worktree` gives an isolated copy of the repo, not a chosen path ([sub-agents](https://code.claude.com/docs/en/sub-agents)). An arbitrary cwd needs a separate `claude -p` process started in that directory.
- **(b) yes.** Order: per-invocation `model`, then frontmatter, then `CLAUDE_CODE_SUBAGENT_MODEL`, then the main model ([sub-agents](https://code.claude.com/docs/en/sub-agents)).
- **(c) partial.** Per-subagent `effort` exists. A per-invocation `effort` parameter needs v2.1.292 or later. "There is no per-subagent thinking setting": subagents inherit the session's extended-thinking on/off ([sub-agents](https://code.claude.com/docs/en/sub-agents)).
- **(d) yes.** `tools` is an allowlist and `disallowedTools` is a denylist. Background subagents keep only a fixed built-in set, which includes `Read`, `Grep`, `Glob`, `Bash`, `Edit`, and `Write` ([sub-agents](https://code.claude.com/docs/en/sub-agents)).
- **(e) partial.** `skills` preloads skills by name from discovery locations, not by path ([skills](https://code.claude.com/docs/en/skills)). `--add-dir` loads skills from `.claude/skills` in an added directory ([CLI reference](https://code.claude.com/docs/en/cli-reference)). A subagent with `Read` can open `<path>/SKILL.md` and resolve relative files itself (inference).
- **Parallel.** Background subagents "run concurrently while you continue working." Background is the default with fork mode, which is on by default in interactive sessions ([sub-agents](https://code.claude.com/docs/en/sub-agents)).
- **Visible.** Foreground subagents block. Background rows appear in the subagent panel and `/tasks`, and the transcript shows a tool-call row with the subagent name ([sub-agents](https://code.claude.com/docs/en/sub-agents)). Headless `-p` visibility is unknown (Q5).
- **Headless flags.** `claude -p` accepts `--model`, `--effort`, `--tools`, `--agent`, and `--add-dir` ([CLI reference](https://code.claude.com/docs/en/cli-reference)).
- **Lifecycle.** A background command started by a foreground subagent stops when that subagent ends ([tools reference](https://code.claude.com/docs/en/tools-reference)).

## Codex CLI

- **Spawn and config.** Built-in agents are `default`, `worker`, and `explorer`. Custom agents are TOML files in `.codex/agents/` or `~/.codex/agents/` with `name`, `description`, and `developer_instructions` ([subagents](https://developers.openai.com/codex/subagents)).
- **(a) partial.** Subagents inherit the parent's cwd. Requests for a per-spawn `cwd` are open: [#18969](https://github.com/openai/codex/issues/18969) (opened 2026-04-22) and [#23095](https://github.com/openai/codex/issues/23095). [#33144](https://github.com/openai/codex/issues/33144) reports named subagents ignoring the requested worktree. For a fixed cwd, use `codex exec --cd <dir>` ([CLI reference](https://developers.openai.com/codex/cli/reference), search snippet; the page did not fetch cleanly).
- **(b) yes.** `model` per agent file or per spawn. Default via `agents.default_subagent_model` ([subagents](https://developers.openai.com/codex/subagents), [config reference](https://developers.openai.com/codex/config-file/config-reference)).
- **(c) yes.** `model_reasoning_effort` per agent file or per spawn. Levels depend on the model ([config reference](https://developers.openai.com/codex/config-file/config-reference)).
- **(d) partial.** Custom agent files cannot override `sandbox_mode`. They inherit the parent's permissions. Instructions "don't create a separate read-only sandbox." Feature keys such as `features.shell_tool` can be disabled ([subagents](https://developers.openai.com/codex/subagents)). A read-only child needs `codex exec --sandbox read-only` ([non-interactive](https://developers.openai.com/codex/noninteractive)).
- **(e) partial.** Skills load from `$CWD/.agents/skills` up to the repo root, `$HOME/.agents/skills`, `/etc/codex/skills`, and system locations. The skill list includes each file path, and full `SKILL.md` loads when selected ([skills](https://developers.openai.com/codex/skills)). Subagents inherit skills ([subagents](https://developers.openai.com/codex/subagents)). `skills.config` only enables or disables by path ([config reference](https://developers.openai.com/codex/config-file/config-reference)).
- **Parallel.** The parent waits until all requested results are available. The cap is `agents.max_concurrent_threads_per_session`, and when unset Codex chooses it ([config reference](https://developers.openai.com/codex/config-file/config-reference)). The older "6" default is no longer stated.
- **Visible.** Open a subagent thread from the main thread, and `/agent` switches threads ([subagents](https://developers.openai.com/codex/subagents)). Visibility under `exec` is unknown (Q1).
- **Timeouts (G2).** `background_terminal_max_timeout` defaults to 300000 ms and is the poll window for empty `write_stdin` polls, not a command timeout. `tool_timeout_sec` (60 s) covers MCP tools only ([config reference](https://developers.openai.com/codex/config-file/config-reference)). The default shell command timeout is not in these pages (Q1).
- **`codex exec`.** Has `--json`, `-o`, `--output-schema`, and `--sandbox`. The default sandbox is read-only ([non-interactive](https://developers.openai.com/codex/noninteractive)).

## opencode

Two doc sets disagree. The stable docs use a `task` tool and `permission.task`. The v2 docs use a `subagent` tool. Which one ships is unknown (Q2).

- **Spawn.** Stable: child sessions launched by the `task` tool, with `permission.task` controlling which agents a parent may launch ([agents](https://opencode.ai/docs/agents/)). v2: the `subagent` tool takes an agent ID, description, and prompt. `background: true` returns immediately and notifies the parent. `sessionID` resumes the child ([v2 tools](https://opencode.ai/v2/docs/tools)).
- **(a) unknown.** `opencode run --dir` sets the directory for a CLI run ([CLI](https://opencode.ai/docs/cli)). No subagent cwd is documented. The shell tool's `workdir` is per call ([v2 tools](https://opencode.ai/v2/docs/tools)).
- **(b) partial.** The model comes from the agent's `model` (`provider/model`, optional `#variant`), otherwise the parent's ([v2 agents](https://opencode.ai/v2/docs/agents)). The `subagent` tool takes no per-call model, so a different model needs a different agent file.
- **(c) yes, provider-dependent.** `reasoningEffort` passes through as a provider option ([agents](https://opencode.ai/docs/agents/)). `opencode run --variant` sets provider-specific reasoning effort ([CLI](https://opencode.ai/docs/cli)).
- **(d) yes.** Per-agent `permission` rules with `edit: deny` and `bash: deny` ([agents](https://opencode.ai/docs/agents/)). The v2 page has a read-only reviewer example ([v2 agents](https://opencode.ai/v2/docs/agents)).
- **(e) partial.** The `skill` tool loads a skill by ID and returns bundled resources ([v2 tools](https://opencode.ai/v2/docs/tools)). Discovery covers `.opencode`, `.claude`, and `.agents` skill folders, walking up from cwd ([skills](https://opencode.ai/docs/skills)). Loading by path is undocumented.
- **Parallel.** The stable docs say the general subagent can "run multiple units of work in parallel" ([agents](https://opencode.ai/docs/agents/)). v2 has `background: true`. No concurrency cap is documented (Q3). Nesting depth defaults to 1 and is set by `subagent_depth` ([config](https://opencode.ai/docs/config)).
- **Visible.** Child sessions are navigable with `session_child_first` and related keys ([agents](https://opencode.ai/docs/agents/)).
- **Shell.** Foreground default is 2 minutes. `timeout` is in milliseconds. A `background: true` call has no timeout unless one is passed ([v2 tools](https://opencode.ai/v2/docs/tools)).

## Pi

- **No built-in subagents.** The README says Pi "skips features like sub-agents" ([README](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/README.md)). Children are separate processes: `--print`, `--mode json`, or `--mode rpc` ([CLI integration](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli-integration.md)).
- **(a) yes.** The working folder is the process cwd, and relative paths resolve from it ([CLI](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md)). Sessions are grouped by cwd ([sessions](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/sessions.md)).
- **(b) yes.** `--model <provider/id>` with an optional `:<thinking>` suffix ([CLI](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md)).
- **(c) yes.** `--thinking <level>` takes `off` through `max`, overrides the suffix, and is clamped to the model ([CLI](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md)).
- **(d) yes.** `--tools read,grep,find,ls` replaces the whole tool set. It does not remove MCP tools ([CLI](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md)). Pi has no sandbox ([security](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/security.md)).
- **(e) yes.** `--skill <path>` loads a skill file or directory, repeatable. `--no-skills` disables discovered skills, and explicit paths still load ([CLI](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md)). Pi tells the model where the skill lives so it can resolve relative paths ([skills](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/skills.md)).
- **Parallel.** Separate processes (inference). Print mode runs once and exits. JSON mode writes JSONL to stdout, with diagnostics on stderr ([CLI integration](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli-integration.md)).
- **Visible.** Only in a pane. Herdr's `agent start --kind` supports `pi`, `claude`, `codex`, and `opencode` ([Herdr](https://herdr.dev/docs/agent-automation/)). tmux packages exist, such as `pi-tmux-subagents` and `pi-extension-subagent`, but I did not verify their source.
- **Experimental.** The checkout has subagent code under `src/experimental`. Whether it is on by default is unverified (Q4).

## Gap status

- **G1 (Codex).** Closed as "no": `sandbox_mode` cannot be overridden per agent, and cwd and skill-by-path are unsupported.
- **G2 (Codex timeouts).** Partly closed: the 5-minute poll window is confirmed. The shell default is open (Q1).
- **G3 (opencode).** Effort is closed (yes). cwd, skill-by-path, and concurrency remain unknown. The `bash` maximum was not rechecked.

## Consequences for fede

The task item must contain:

1. An absolute `cwd`.
2. A provider-neutral model ID, mapped per harness: Claude alias or ID, Codex model, opencode `provider/model`, Pi `provider/id`.
3. An effort level, mapped per harness: Claude `effort`, Codex `model_reasoning_effort`, opencode `--variant`, Pi `--thinking`.
4. An abstract tool profile such as `read-only`, mapped per harness: Claude `tools`, opencode permissions, Pi `--tools read,grep,find,ls`. Codex has no per-agent sandbox, so use `codex exec --sandbox read-only` or accept inherited permissions.
5. Absolute paths for the prompt file and the result schema. Only Codex `exec` enforces a schema (`--output-schema`), so fede should validate on `fede submit`.
6. Absolute `SKILL.md` paths, with the instruction to resolve relative files against the skill's directory. This is the only path-independent method on Claude, Codex, and opencode.
7. A deadline. Do not assume a foreground shell command lasts longer than 2 minutes (Claude and opencode defaults).

**Uniform fallback.** Launch each task as a shell child, which uses flags the sources document: `codex exec --cd <dir> --model <m> -c model_reasoning_effort=<e> --sandbox read-only --output-schema <s> -o <out>`; `claude -p --model <m> --effort <e> --tools <list> --add-dir <dir>` from the target cwd; `opencode run --dir <dir> -m <m> --variant <v> --agent <a>`; `pi --print --model <m>:<e> --tools <list> --skill <path>`. The cost is that native visibility is lost unless the child runs in a pane. opencode's tool profile needs a pre-written agent file.

**fede cannot rely on:**
- A Codex subagent running in another cwd.
- Read-only enforcement for a Codex subagent beyond instructions.
- A per-subagent thinking setting on Claude.
- opencode subagent cwd, skill loading by path, or a concurrency cap.
- Automatic loading of an arbitrary skill path on Claude, Codex, or opencode. Only Pi's `--skill` does this.
- Stable tool names in opencode (`task` or `subagent`).
- Visibility of headless children.

## Unknowns

- **Q1.** Codex default shell command timeout. Whether `exec` subagent activity is visible.
- **Q2.** Which opencode doc set matches shipped releases (`task` or `subagent`).
- **Q3.** Whether opencode subagents inherit cwd, can reach skills by path, and have a concurrency cap.
- **Q4.** Whether Pi's experimental subagent is enabled by default.
- **Q5.** Whether Claude `-p` background subagents are visible anywhere.
- **Q6.** Whether Codex will add a per-spawn cwd (#18969 is open).
- **Q7.** The pages are undated. Re-check versions before shipping.

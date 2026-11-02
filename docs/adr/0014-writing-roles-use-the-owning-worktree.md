# Writing roles work in the branch's owning worktree

During a run, the branch is written only in its owning worktree, the worktree that has the branch checked out. fede finds it with `git worktree list`. fede's own worktree is a detached checkout at the reviewed commit: it holds review input, the repo `.fede/` layer and check runs, and it never writes. Every role that writes the branch works in the owning worktree: the fixer, the conflict resolver and the `--commit` at handover. This keeps slice 1, where the human or implementer commits fixes in their own worktree, and the later fixer on one rule. We rejected moving the branch into fede's worktree because git allows one checkout per branch, so the implementer would have to release the branch at handover and check it out again for every manual fix. Claude Code and Codex subagents also inherit the coordinator's cwd, which is usually the implementer's worktree. We rejected having the fixer commit on fede's detached HEAD and then moving the branch ref, because that leaves the owning worktree out of sync with its branch.

## Rules

- **No owning worktree.** When no worktree has the branch checked out, fede adds one under the run directory and removes it at done or stop. fede never takes the branch from an existing worktree.
- **Dirty owning worktree.** When a writing task is due and the owning worktree has uncommitted changes, `next` returns a `wait` on `clean-worktree` listing the dirty paths, and re-checks on every call. fede never stashes or discards the user's changes.
- **The cwd is rendered, not sent.** fede resolves the owning worktree when it issues the task and writes its absolute path into the rendered prompt file, with the instruction to work only there. Items gain no field (ADR 0006).
- **Submit reads the branch ref.** A writing task's submit requires at least one new commit on the branch that descends from the head the task was issued at, and a clean owning worktree. fede records the new branch head as the result commit.
- **One writer per branch is a convention.** fede cannot stop another writer. Any head movement fede did not ask for is a new head: results at the old head go stale (ADR 0005) and the next round reviews the new head. The fede skill tells implementers not to commit during a run except in answer to a fix wait, and tells coordinators to run from the branch's owning worktree.

## Considered options

- **fede's worktree owns the branch:** rejected for the handover cost and the cwd inheritance above.
- **Fixer commits on a detached HEAD and fede moves the ref:** rejected, it desyncs the worktree that has the branch checked out.
- **Escalating every unrequested head movement:** rejected, it adds decisions and still cannot catch a write mixed into a fixer's own commit.
- **Stashing a dirty owning worktree:** rejected, fede does not touch uncommitted user work.

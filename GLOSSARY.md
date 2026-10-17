# fede

fede takes a change an agent has implemented and drives it to a merge-ready draft pull request: review, verification, repair, CI and conflicts, calling the human only for decisions or the final approval.

## Runs and the coordinator

**Run**:
One pass of fede over one branch, from the moment the change is handed over until it is done or stopped.
_Avoid_: job, session, pipeline run

**Coordinator**:
The agent, inside the user's harness, that calls `fede next`, carries out what it returns, and submits the results.
_Avoid_: supervisor, driver, orchestrator

**Item**:
The single thing `fede next` hands the coordinator: a task, a decision, a wait, or done.
_Avoid_: step result, instruction

**Task**:
An item asking the coordinator to have an agent do one bounded job in a named role and return a result.
_Avoid_: job, work item

**Decision**:
An item that only the human can resolve, such as a finding touching the spec or a gated push.
_Avoid_: approval, question

**Wait**:
An item telling the coordinator that nothing can progress until something outside changes, and when to ask again.

**Role**:
The job an agent performs in a task: reviewer, verifier, fixer, conflict resolver, or coordinator.

**Step**:
A stage of the fixed pipeline, such as review, checks or push.
_Avoid_: stage, gate

## Findings

**Finding**:
A claimed problem with the change, from a reviewer, a failed check, CI, a conflict, or an external review comment.
_Avoid_: issue, comment, defect

**Ledger**:
The record of every finding across every round of a run, including fixed, dropped and deferred ones.
_Avoid_: report, findings list

**Disposition**:
What happens to a finding: fix, drop, or defer.

**Tripwire**:
A deterministic rule that forces a finding to the human regardless of any agent's judgment, such as a change to an exported API or a spec file.

**Handoff**:
The file listing findings the human chose to fix, written so any agent can pick it up.

**Escalation**:
Turning a finding or blocker into a decision for the human.

## Verification

**Check**:
A verification fede runs or requests: a command, an agent driving a verification skill, or a forge check such as CI.
_Avoid_: test, gate

**Evidence**:
Artifacts that prove a check's verdict, bound to the commit they were produced at.
_Avoid_: proof, logs

**Verdict**:
A check's result at a commit: pass, pass with notes, or fail.

**Budget**:
The number of repair attempts fede allows in a category (review fixes, check fixes, conflict repairs) before escalating.

## Delivery

**Push plan**:
What a push would publish: its commits, the findings it addresses and the replies it posts. It is computed, not stored.

**Ready packet**:
The state in which fede hands the change to the human: CI green, no conflicts, review clean, evidence attached, acceptance criteria met, pull request still a draft.

**HITL mode**:
The default mode, in which the human approves the ready packet before the pull request leaves draft.

**YOLO mode**:
The mode in which fede also marks the pull request ready for review on its own. It never merges.

## Customization

**Package**:
A directory of prompts, schemas, policies and vendored skills that shapes how fede reviews and verifies, possibly extending another package.
_Avoid_: plugin, profile, preset

**Layer**:
One directory in the chain fede searches for prompts and skills: the repo's `.fede/`, the project profile, a package, or a package it extends.

**Override**:
A prompt file or skill directory in a higher layer that replaces the one at the same path in a lower layer.
_Avoid_: patch, fork

**Reference package**:
The package shipped with fede, opinionated and usable out of the box.

**Tier**:
A named class of model (fast, medium, flagship, frontier) that a task asks for; the coordinator maps it to a concrete model.
_Avoid_: model category, model class

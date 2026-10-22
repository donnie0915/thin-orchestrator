# <Task ID>: <reviewable deliverable>

## Assignment
Executor: flash-builder
Phase: <phase ID>
Working directory: <exact path verified by the main session>
Baseline: <branch/commit + pre-existing changes, or an explicit non-Git snapshot>
Dependencies: <accepted task IDs and their concrete artifacts in this workspace>
Report / checkpoint path: <path owned exclusively by this task>

## Goals and non-goals
<A complete, coherent bundle — not a tiny step.>

## Done criteria
- <Observable behavior and its acceptance ID.>
- <Negative or edge behavior.>
- <Required verification evidence.>

## Context and exact contract
<Only the relevant repo patterns, types, error cases, and interface signatures.>

## Files and ownership
May modify: <specific files/directories, including this task's tests and report.>
Must not modify: <other workers' scopes, unrelated user changes.>
Excluded by default: secrets/.env files, production config, undeclared dependencies or lockfiles, CI, unrelated
migrations, Claude/editor config, .git internals.
Explicit exceptions: <none, or specific exceptions authorized by the user.>

## Implementation freedom
<Important internal implementation choices left to the worker. Don't pre-write every function.>

## Verification
Working directory: <path>
Commands and expected results:
- `<actual command>` -> <expected result>
Required test cases: <exact normal, negative, and edge cases.>
UI/runtime checks: <observable states, or why not applicable.>

## Budget, checkpoints, and stop rules
<A run budget and meaningful milestones suited to this task; don't invent a generic duration.>
Autonomously continue the in-scope test/code/fix loop. When a contract is missing, there is an unsafe conflict, the
environment is unavailable, or the same approach keeps failing, report instead of inventing requirements.
No nested subagents/CLIs, no bypassing permissions, no commit/merge/deploy.

## Required return
Follow the task-report.md format. Include actual commands/exit codes, changed files, unresolved risks, and a resume
checkpoint. STATUS is ready_for_review, blocked, or failed; only the main session can accept the task.

# Execution and workspace ownership

## Default: one writer, no extra orchestration

By default, dispatch one flash-builder in the current workspace to handle one complete end-to-end phase bundle.
The main session provides the contract, dispatches once, waits once, and reviews one completion report. The worker
owns in-scope repo exploration, implementation, testing, debugging, and any routine browser/visual QA the brief
requires. No external CLI, script runner, or auto-commit is needed.

Record a baseline before dispatching: workspace root, current branch/HEAD (if Git), tracked/staged diff, list of
untracked files. When you need to tell worker changes apart, save the relevant pre-existing file contents to a safe
local baseline.
Do not read or copy secrets to snapshot the workspace. Never auto-stash, reset, commit, or discard the user's changes.
Avoid assigning unrelated files the user is actively editing — a list of filenames alone cannot restore those
changes; either negotiate a safe baseline or pick an independent scope.

Tell the worker what state it should see:

- Shared workspace: uncommitted changes are visible and must be preserved.
- `isolation: "worktree"`: the worktree is created from HEAD, so **uncommitted changes in the parent workspace are
  not in it**. Check the files actually produced; don't rely on "the contract is already shared".

While the worker owns certain paths, the main session makes no overlapping edits. The worker does not edit shared
planning/state files assigned to the coordinator; give its report a unique output path. Role instructions are not
filesystem permissions: the host's permission settings are authoritative.

## Optional: two truly independent writers

Parallelism must be declared item by item in the plan. Default to at most two writers, and only when: dependencies
are satisfied, write scopes do not overlap, and the second writer uses `isolation: "worktree"` to get a genuinely
separate workspace. Merely being another subagent thread is not isolation.
Shared types, dependency manifests/lockfiles, routes, migrations, generated files, and schema files are usual
conflict points — handle them serially.

With worktrees, record the base commit per task in the plan/report. Do not secretly commit a dirty base just to make
creating a worktree convenient. Worktrees do not isolate processes, credentials, ports, or network.

Review and integrate each result before dependent tasks start; run one combined check after integration — branches
that pass separately can still conflict when combined.
Keep worktrees/branches until review is done; never auto-delete a worktree with changes or an unmerged result.

## Handoff, waiting, and rework

Give the task its full brief and relevant shared context. The worker **cannot see** this session's conversation
history; never tell a fresh worker to "continue what we just discussed" without attaching the concrete decisions.

How to wait: the Agent call runs in the background and notifies on completion. Just wait for the notification:

- Do not read its output/transcript file, poll, ask for progress, interrupt a healthy run, or duplicate its repo work.
- While waiting, you may do non-overlapping work (e.g. drafting the next phase's brief) or simply end the turn.
- A long wait does not by itself mean the worker is stuck.

When the worker is genuinely blocked, the main session resolves the contract or environment issue, then sends a
targeted update to the same worker via SendMessage.
For rework, collect all concrete issues and send them in one SendMessage to the same worker (keeping its existing
context). Never dispatch a new worker for the same bundle.
Default to one rework round. Re-check only the affected behavior and accepted dependencies; widen scope only for
material high risk.
Start a new worker only for a new bundle unrelated to this task.

## Git and production boundaries

This flow does not include automatic commit, staging, branch merging, push, deploy, release, or production
migrations. Those happen only with the user's explicit authorization, and only after actual review.

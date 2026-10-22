---
name: thin-orchestrator
description: Use for larger multi-file features, refactors, migrations, or executing an existing plan. The main session (Opus or Fable) owns design, task briefs, and final acceptance; the flash-builder subagent (Sonnet) owns repo exploration, implementation, testing, and debugging. Skip for one-line fixes, pure Q&A, or when the user explicitly asks not to delegate. The flash-builder subagent itself must never use this skill.
---

# Main session plans, flash-builder implements, main session accepts

This skill defines a workflow only. It does not care which model runs the main session (Opus or Fable both work;
switch with `/model`). The worker's model comes from the `model` field of the `flash-builder` agent definition
(`agents/flash-builder.md`).

## Shape of the flow

Spend main-session tokens only where leverage is highest: architecture, acceptance criteria, material risk, and
final acceptance. Once the contract is set, flash-builder owns in-scope repo exploration, implementation, testing,
debugging, and routine browser/visual QA.

A normal phase looks like this in the root session: **one plan → one dispatch to one worker → one wait → one batched
review → one final reply**. This is a shape, not an excuse to skip evidence or to stop before the result is right.
Only a concrete blocker, or architecture/security/production risk, adds main-session work.
Never generate root-session activity just to "check progress".

## 1. Size the work

Read the relevant repo guidance and the current request, and keep existing work intact. Decide whether this is a
direct small fix, a bounded build, or a large multi-phase project.
Pass this gate before dispatching. **Dispatch a worker only if both hold:**

- The change is expected to touch 3+ files, or to create a complete new module/page;
- Implementation needs **repeated** cycles of reading repo files, running tests, reading errors, and fixing — not a
  single write-and-hand-in.

If either condition fails, the main session does the work directly and skips this delegation flow. Local measurement:
for a ~500-line, 2-file task written in one pass, delegating cost 1.9× as much as doing it in the main session and
used 2.9× the tokens. The fixed overhead of delegation (a fresh worker context of ~50k tokens, writing the brief,
reviewing the whole patch) ate the entire unit-price advantage of the cheaper model. Savings only happen when the
implementation loop itself is long.

For larger builds, state up front what the main session owns and what flash-builder will implement.

Carry forward the user's existing authorizations and decisions. If the user explicitly asked to "plan and implement",
the flow within that scope is authorized; do not ask again at every phase. Only ask about material product/risk
decisions the repo cannot answer. If the user only asked for a plan, do not implement.

## 2. Confirm the worker is available

Confirm that `flash-builder` is among the Agent tool's available types. When installed as a plugin it may appear
namespaced, e.g. `thin-orchestrator:flash-builder`; use whichever name is listed. If it is missing, finish the plan,
stop, and report why.
**Do not** substitute general-purpose or any other type, and do not pass a `model` override in the Agent call:
other types inherit the main session's model by default, which defeats the whole cost premise.

## 3. Design first, then split

Read [references/planning.md](references/planning.md). Reuse an existing approved design/spec and phase plan if there
is one (Superpowers, GSD, or the project's own plans all count). Do not write a competing spec and do not force a
framework migration.

For new work, write a design sized to the problem: goals, non-goals, repo evidence, meaningful alternatives,
interfaces, failure behavior, risks, acceptance criteria. Shared contracts come before the implementations that
depend on them. Architecture, auth/security, multi-tenancy, payments, secrets, and production-affecting decisions
are made by the main session — never tossed to the worker as a vague "just build it".

## 4. Phase plan and dispatchable briefs

Default artifact directory: `docs/agent-work/<feature>/` (follow the project's convention if it has one). Use the
templates as needed: [spec](templates/spec.md), [task-brief](templates/task-brief.md),
[task-report](templates/task-report.md), [checkpoint](templates/checkpoint.md).

Write a dependency-ordered phase plan. When contracts are stable, prefer one end-to-end vertical bundle per phase.
Each worker task needs: a precise contract, a bounded file scope, a testable outcome, and verification commands.
Internal exploration, implementation, testing, debugging, and routine UI verification are left to the worker; the
main session does not write the full implementation into the plan.
Long tasks inside a clear contract are fine, but they must not cross unknown architectural boundaries.

Split only at real dependency or independent-acceptance boundaries — not for progress visibility, not per function,
not per five-minute step. Finish earlier phases first, and refine later briefs once the earlier interfaces settle.

## 5. Dispatch, then let go

Read [references/execution.md](references/execution.md).

- Record a baseline before dispatching: `git status --porcelain`, branch/HEAD, list of untracked files (for non-Git
  directories, snapshot the relevant files).
- Use the Agent tool with the flash-builder agent type. Put the full brief in the prompt plus the minimum shared
  context: contract, task ID, working directory, baseline, dependency artifacts, report path. The worker **cannot
  see** this session's history, so every decision must be in the brief.
  Do not forward the whole conversation, redo the repo investigation, or forward long logs.
- By default there is exactly **one** writer in the current workspace. The main session does not edit paths the
  worker owns. A second writer is allowed only for tasks the plan explicitly marks as independent with
  non-overlapping write scopes, and it must use `isolation: "worktree"`.
- After dispatching, wait for the completion notification. Do not read its output file, poll, ask for progress,
  interrupt a healthy run, or do overlapping repo work. You may do non-overlapping work while waiting, or simply end
  the turn. A long wait does not by itself mean the worker is stuck.
- Rework or additional information **must** go to the same worker via SendMessage. **Never dispatch a new worker for
  the same task bundle**: a new worker pays the ~50k-token fixed context cost again and loses the understanding the
  previous round built. Only start a new worker for a genuinely unrelated bundle.

## 6. Review the actual result

Read [references/review.md](references/review.md). A finished worker means **pending review**, not accepted.
Review the actual patch against the recorded baseline, including new untracked files and pre-existing changes.
Do not treat HEAD as the baseline before you have checked the workspace state.

Apply two lenses in **one** batched review: spec compliance, then code quality/security. Look at the actual diff and
the worker's evidence. Spot-check only when evidence is missing, failure is plausible, or risk warrants independent
confirmation. Do not routinely rerun the whole test suite, and do not repeat visual QA that already has artifacts.
Choose spot-checks from cheapest to most expensive: read the worker's evidence → run the verification commands it
gave → read the key code.
**The main session must not write its own verification code, reference implementations, or comparison scripts** —
that is implementation work, and writing throwaway scripts at the main session's unit price is the most expensive
option. If an independent implementation really is needed for cross-checking, write it as a brief for the worker;
the main session only judges the comparison.

Accept only when both lenses pass and the relevant verification succeeds. When changes are needed, send every
file-level issue to the same worker in one message. Default to at most one rework round, then accept, or explicitly
reassess scope and risk. A second round is reserved for material open issues under "High-risk exception" below.

## 7. Integrate, checkpoint, wrap up

The main session integrates accepted dependencies in order and respects the user's existing Git permissions.
Finishing a phase is not a reason to automatically commit, merge, push, deploy, or run production migrations.
Changes in a shared workspace are already in the workspace; do not invent a branch merge.

Run cross-task checks only at real integration boundaries, and only when they cover more than the worker's
already-accepted evidence.
At meaningful resumable boundaries (not every turn), update the plan and a short `CHECKPOINT.md`: accepted work,
current diff/workspace, contract decisions, pending tasks, worker names/IDs, review status, exact resume action.

The final reply states what was done, what actually passed, and remaining limitations. Do not estimate savings from
task count or elapsed time; to measure cost, use actual usage records.

## High-risk exception

Expand main-session investigation, checks, or rework rounds only when evidence shows material risk: architecture,
authentication/authorization, payments, multi-tenancy, secrets, destructive migrations, production behavior,
high-impact shared infrastructure. State the reason when you expand.
High risk is not a reason to poll progress, duplicate implementation, or routinely rerun things.

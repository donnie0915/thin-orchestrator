# Planning: don't implement twice

## Pick the process by uncertainty

- One-line fix: no design doc, task list, or worker.
- Bounded feature: a short design + one task brief + review evidence.
- Vague app/rebuild: discover first, then write a spec, phase by dependency, and make integration checks explicit.

Documentation scales with uncertainty, not with how big the task's name sounds.

If Superpowers / GSD / the project already has a spec and plan, read its actual content and reuse its terminology,
requirements, and settled decisions. Only add the execution mapping — "what goes to flash-builder, how it is
accepted" — instead of starting a competing plan.
Never pretend a skill that doesn't exist has already run.

## Design work owned by the main session

Pin down: user-visible outcome, out-of-scope work, constraints, existing repo patterns, uncertain assumptions.
Compare alternatives only where a real decision exists. Settle data ownership, component boundaries, interface
signatures, error states, and backward compatibility.
Define testable behavior before dispatching, including negative cases. For security-sensitive behavior, the main
session owns threat analysis and core correctness — not just a final glance.

If the user already authorized "plan and implement", there is no need for a formal confirmation step. Material open
product choices, destructive changes, external publishing, spending beyond the approved scope, and production access
are still decisions that need separate confirmation.

## Phases and task bundles

A phase is an integration milestone. A bundle is a unit of work one worker can complete and the main session can
accept or reject independently.
When contracts are stable, prefer one end-to-end vertical bundle per phase. Split only at real dependency,
ownership, risk, workspace, or independent-acceptance boundaries — never to manufacture progress checkpoints.
For example:

- Foundation: establish shared types and test fixtures, including contract tests.
- Implementation: build one complete, bounded behavior on top of those contracts.
- Integration: wire up callers and verify error states and system behavior.

These are examples, not mandatory layers for every project. Working vertical slices beat disconnected scaffolding.
A page with its loading / error / empty states and their tests can be one bundle; splitting every file into its own
subagent usually just adds orchestration overhead.

Write dependency edges explicitly. A task may start only after its prerequisites are accepted and their artifacts
actually exist in its workspace.
Repo paths and example patterns come from actual inspection — never invent them. Unknown future contracts stay as
"open tasks", not dispatchable briefs.
Update the remaining plan when evidence changes; don't send a stale brief just because it was written first.

## A dispatchable brief

Contains: desired behavior, non-goals, relevant context, exact interface shapes, paths allowed to change, things
that must not change, test cases, verification commands, prerequisites, completion report format, stop conditions.
For tricky contracts, a short example or pseudocode is fine.
Do not write the full implementation or dictate every line; leave implementation freedom to the worker within the
contract.

A long worker run should mean "steadily executing one clear task", not "an unbounded autonomous project".
Require a checkpoint if interrupted or blocked; otherwise expect one completion report after its internal
test/fix/UI-verification loop ends.
A large context window is not a reason to stuff every file and every past message into it.

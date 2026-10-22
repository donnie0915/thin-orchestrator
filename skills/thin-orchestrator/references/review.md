# The main session's acceptance gate

Treat the worker's report as claims that need evidence. Review the files actually changed against the task's recorded
baseline, including new untracked files.
Don't blame the worker for pre-existing changes unrelated to the task; and don't miss new files just because
`git diff HEAD` doesn't show them (use `git status --porcelain` + `git diff` + inspect each new file).

## Lens one: spec compliance

Map each acceptance criterion to code, tests, or observed behavior. Check contract names/types, failure cases, edge
conditions, compatibility, and declared non-goals.
Look for: missing behavior, out-of-scope work, placeholder implementations, invented requirements. Confirm the tests
actually exercise the target behavior rather than a stub.
Flag acceptance criteria the environment cannot verify.

## Lens two: quality and risk

Look for: logic bugs, unintended data flow, security/auth mistakes, unsafe concurrency, lifecycle/cleanup issues,
poor error handling, brittle test mocks, weakened types, unnecessary dependency or config changes.
Follow existing repo conventions, but don't demand gratuitous rewrites.
Review all code changed by this task; don't paste the entire patch into the conversation as a ritual.

Both lenses happen in **one** batched review — not two tool loops, and not two more subagents.
When high-risk work genuinely needs an independent reviewer, explicitly choose the reviewer's model and permissions
and justify the cost.
Note: a general-purpose subagent inherits the root model by default, so dispatching it to review means paying the
main session's price.

## Verification and decision

Start from the worker's concrete evidence: commands, exit codes, key output, visual artifacts. Inspect the files
actually changed.
Independently rerun **targeted** checks only when: evidence is missing, failure is plausible, integration introduces
new risk, or high risk needs confirmation.
Don't routinely rerun the whole suite or repeat browser QA. For checks blocked by the environment, record exactly
what was not verified.
Verification methods differ sharply in price; choose from cheapest to most expensive: read the evidence → run
existing commands → read key code → have the worker add tests.
The main session writing its own reference implementation, comparison script, or one-off verification tool is the
most expensive of all and is forbidden by default; when truly necessary, dispatch it to the worker and have the main
session only interpret the results.
A requirement without verification cannot be claimed as accepted.

There are only three outcomes: accepted, changes requested, blocked. Only the main session can decide accepted.
Send all file-level issues and expected fixes to the same worker in one SendMessage.
Re-check the affected fixes; run phase-level integration checks only at real integration boundaries. Default to one
rework round; a second round needs a material high-risk justification.
Isolated tests passing does not mean cross-task compatibility or overall completion.

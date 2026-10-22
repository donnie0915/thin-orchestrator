# thin-orchestrator

English | [简体中文](README.zh-CN.md)

**Your expensive model thinks. Your cheap model types.**

A [Claude Code](https://claude.com/claude-code) skill + subagent that splits big coding tasks between two tiers:

- **Main session** (Opus or Fable) — designs, writes the task brief, reviews the result, and decides whether to accept it.
- **`flash-builder` subagent** (Sonnet) — explores the repo, implements, runs tests, debugs, and hands back one report.

The main session's tokens go only where they matter most: architecture, acceptance criteria, real risk, and the
final verdict. The long, noisy edit → test → fix loop runs on the cheaper model.

## How it works

```
 you ──▶ main session (Opus/Fable)
           │ 1. size the work — is delegation even worth it?
           │ 2. design + write one precise brief
           │ 3. dispatch ONE flash-builder, then wait (no polling)
           ▼
         flash-builder (Sonnet)
           │ explore → implement → test → fix → … → report
           ▼
         main session
           │ 4. one batched review: spec compliance + quality/security
           │ 5. accept, or send all fixes back to the SAME worker (max ~1 round)
           ▼
         final reply: what was done, what actually passed, what's left
```

A normal phase is: **one plan → one dispatch → one wait → one review → one reply.**

## When it delegates — and when it doesn't

Delegation has a fixed cost: a fresh worker context (~50k tokens), writing the brief, and reviewing the whole patch.
So the skill only delegates when **both** hold:

1. The change touches **3+ files**, or creates a whole new module/page; and
2. Implementation needs **repeated** read → test → fix cycles, not a single write.

Otherwise the main session just does it. In one local measurement, delegating a ~500-line, 2-file, write-once task
cost **1.9×** more and used **2.9×** the tokens of doing it directly. Savings only show up when the implementation
loop itself is long.

## Key rules

- **One writer per workspace.** A second parallel writer only for explicitly independent tasks, in a git worktree.
- **Hands off while waiting.** No polling, no progress checks, no overlapping edits.
- **Rework goes to the same worker** via `SendMessage` — never spawn a fresh worker for the same bundle (you'd pay
  the context cost again and lose what it learned).
- **Done ≠ accepted.** Only the main session can accept. Review the actual diff against a recorded baseline,
  including untracked files.
- **Cheap verification first:** read evidence → run the worker's commands → read key code. The main session does not
  write its own reference implementations or comparison scripts.
- **No substitutes.** If `flash-builder` isn't available, the skill stops after planning instead of falling back to
  `general-purpose` (which would inherit the expensive model).
- **No automatic Git or production actions.** No commit, merge, push, deploy, or migrations without explicit approval.
- **High-risk exception.** Auth, payments, secrets, multi-tenancy, destructive migrations, and production behavior
  get more main-session scrutiny — with a stated reason.

## Install

### As a plugin (recommended)

In Claude Code:

```
/plugin marketplace add donnie0915/thin-orchestrator
/plugin install thin-orchestrator@thin-orchestrator
```

When installed as a plugin, the subagent may show up namespaced (e.g. `thin-orchestrator:flash-builder`); the skill
handles either name.

### Manually

```sh
git clone https://github.com/donnie0915/thin-orchestrator.git
mkdir -p ~/.claude/skills ~/.claude/agents
cp -r thin-orchestrator/skills/thin-orchestrator ~/.claude/skills/
cp thin-orchestrator/agents/flash-builder.md ~/.claude/agents/
```

Restart Claude Code (or start a new session) so the skill and agent are picked up.

## Usage

Just describe a larger task. The skill triggers on multi-file features, refactors, migrations, or "implement this
plan" requests, for example:

> Plan and implement CSV export for the orders page, with tests.

You can also invoke it explicitly with `/thin-orchestrator`. To keep everything in the main session, say so
("don't delegate").

To change the worker model, edit the `model` field in `agents/flash-builder.md` (`sonnet`, `haiku`, …).

## Layout

```
.claude-plugin/
  plugin.json            plugin manifest
  marketplace.json       lets this repo act as its own marketplace
agents/
  flash-builder.md       the implementation worker (model: sonnet)
skills/thin-orchestrator/
  SKILL.md               the workflow
  references/            planning.md, execution.md, review.md
  templates/             spec, task-brief, task-report, checkpoint
```

Work artifacts default to `docs/agent-work/<feature>/` in your project, unless the project already has its own
convention.

## Acknowledgements

Inspired by the planner/builder split in
[Astra Flash Orchestrator / Codex on Crack](https://github.com/ethanplusai/codex-on-crack) (MIT), adapted for
Claude Code's native subagents and tuned for cost.

## License

[MIT](LICENSE)

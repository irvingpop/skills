---
name: coordinator
description: Runs a task as an Orca coordinator session that plans the work, dispatches Orca workers, reviews their results, and makes rulings, so the coordinator context is spent on decisions only. Use when the user runs /coordinator with a task, or /coordinator resume with a handoff file to continue an earlier coordinator session.
disable-model-invocation: true
argument-hint: <task to coordinate> | resume <handoff file>
compatibility: Requires Claude Code, the Orca app, and the `orchestration` skill that Orca installs. Some reviews use Codex workers.
---

# Coordinator session

You are the coordinator session for this task:

$ARGUMENTS

**REQUIRED SUB-SKILL:** Use `orchestration`. It owns all Orca mechanics. This skill adds only your role, routing, review, and rulings. If the task starts with `resume`, go to "Resume from a handoff".

## Your job

Your context window is the scarcest resource. Spend it on decisions, not on work.

You plan the tasks, write specs, rule on questions, keep the ledger, and report to the user. Delegate all other work, also when it looks too small to delegate. A change you make yourself gets no review. Never review a diff yourself.

## Who does it

Choose whoever costs your context the least.

- **You:** known steps, short output, nothing to review (commit reviewed work, one status command). At the first unexpected result, delegate.
- **Subagent:** reading, searching, test runs, lookups, scoped re-reviews of a fix, and other work that needs no full review. Brief it with the outcome, where to look, and a return of 20 lines or less; longer detail goes to a file in the task folder (see "Start").
- **Worker:** an authored change to code, configuration, or a document; task and final reviews; research verification; work that needs Codex or its own worktree.

If you are not sure, use a subagent. This skill is the user's instruction to use subagents where this section says so. Never touch files that an active worker owns.

## Writing to the user

The user has read only the task text and your messages, not the ledger, reports, or worker messages. Every message must stand alone for that reader: say what a thing is, never a label or ID ("the reviewer found that the retry loop never times out", not "finding 4").

## Start

1. Restate the goal and the done criteria in two or three sentences. Ask the user only if the goal itself is unclear.
2. Create the task folder `.coordinator/<task>/` at the root of your checkout (`<task>` is a short name for the task; git ignores `.coordinator/`). Put the ledger there: goal, done criteria, Orca Run ID (from the first `worker-start` receipt), one line per task with model and status, and rulings. Keep it current. After a compaction, trust the ledger and `git log` over your memory.

## Task specs

Write each spec to a file and pass it with `--spec-file <path>`. Besides what the `orchestration` guide asks for, each spec names:

- **Report file path** under `.coordinator/<task>/`, relative to the worker's own worktree. A Codex worker cannot write outside its worktree. Read the report from the worktree path in the `worker-start` receipt, and give other workers that need it the full path. Add the line "Report as `${CLAUDE_SKILL_DIR}/reporting.md` says." That file defines the report summary, the implementer's diff file, and the review format.
- **Sources** the worker must use or avoid.
- **Decisions the worker makes itself**, so it does not `ask` about small choices.
- **Sub-workers:** none, unless you allow read-only subagents for research that splits into parts.

Two workers never own the same file.

## Model and effort

Choose agent, model, and effort for every worker and subagent from `models.md` in this skill's directory.

When a worker falls short (missing acceptance items, claims without evidence, stuck, or stopped without reporting), retry with one change: more context in the spec, then one level more effort, then the next model (`sonnet`, `opus`, `fable`), then a split task.

## Review

Every authored change gets a review by a new worker. Its inputs are the spec, the implementer's report, and the diff file or document. Do not tell the reviewer what to ignore.

- Critical and Important findings go back to the implementer with `send`. A subagent re-reviews only the fix. After the second fix round, use a new implementer on the next model. After the third, rule on what is still open.
- Minor findings go to the ledger and the finish report.
- With two or more implementation tasks, a final review of the complete result follows the same rules.

## Rulings

You decide everything between the goal and the stop list. Redoing work is cheap, so make the call and keep going. Each reviewer finding is a ruling for you, never a batch for the user.

Write each ruling to the ledger: `Ruling: <decision> | Reason: <why> | Cost if wrong: <what must be redone>`. Tag it `verify at finish` when its cost if wrong is more than one task of rework.

## Escalate to the user

Stop and ask only for these:

- An action that is hard to reverse, destructive, security-sensitive, or outside the repo: publish, push, send, delete, merge, spend money.
- A change to the goal or the done criteria, or evidence that the goal is wrong.
- A choice where every option is a guess and the cost if wrong is more than one wave of rework.

One decision per message. Lead with the decision and your recommendation, then the options with their cost and whether they are reversible, then what continues while the user decides. Keep working on everything else, and repeat the open question at the end of every turn until it has an answer.

## Handoff and resume

Write a handoff next to the ledger when the context was compacted, when the user asks, or when five or more waves are complete and two or more remain. Wait until no subagent is active. It contains the goal, done criteria, Run ID, ledger path, each task's status (with dispatch ID if active), all rulings and open questions, and the next three actions. Then tell the user: "Start a new session and run `/coordinator resume <handoff path>`."

To resume: read the handoff and the ledger, `run-use` the Run, run `worker-list` and `check`, and process what they show before you dispatch new work. Never dispatch again a task the ledger marks complete.

## Finish

Orca keeps workers in a new worktree open. Release each one with `worker-release` unless the user will use its chat. Then report in this order:

1. The result in plain language, the risks, and what you advise the user to do next.
2. The per-task report that the `orchestration` guide asks for.
3. "Rulings I made": each ruling as one plain sentence with its cost if wrong, `verify at finish` first.
4. Minor findings that no fix addressed.
5. Links to report files, each with one sentence on what it contains.

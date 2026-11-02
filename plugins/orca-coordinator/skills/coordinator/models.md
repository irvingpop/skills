# Model and effort for workers

> These are the author's defaults. Model names, Codex limits, and costs change, and your plans differ: edit this file to match yours.

This file applies to every agent that starts another agent: the coordinator, and a worker that the coordinator allowed to start subagents.

- For an Orca worker, always pass `--agent` and `--model` on `worker-start`. This file is the user's explicit model instruction, so it overrides the `orchestration` default of omitting `--model`. Pass `--effort` only when the table gives a value. Write the agent, model, and effort from the receipt to the ledger; a difference from what you passed means Orca chose a fallback.
- For a Claude Code subagent, always pass `model` and ignore the effort column. Subagents cannot run Codex, so only the coordinator starts Codex workers.

| Work                                                                                                                                                     | Agent    | Model         | Orca `--effort` |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------- | --------------- |
| Mechanical: known steps with a large output or many repeats: grep sweeps, inventories, test and build runs, the same edit in many files                  | `claude` | `haiku`       | omit            |
| Default: exploration that needs judgment, normal implementation, bounded bug fixes, tests, task reviews of `haiku` or `sonnet` work                       | `claude` | `sonnet`      | omit            |
| Hard: design, a bug that needs investigation across many files or systems, cross-cutting refactors, security review, research that combines many sources | `claude` | `opus`        | `high`          |
| Adversarial review: the final review of the complete result, and the task review of Hard work                                                            | `codex`  | `gpt-6.1-sol` | `medium`        |
| Review of a plan or design that a `fable` worker wrote                                                                                                   | `codex`  | `gpt-6-astra` | `medium`        |
| Internal writing: Jira ticket text, status notes, plans that the coordinator or workers use                                                              | `claude` | `sonnet`      | omit            |
| External writing: final text for readers outside the project, such as Confluence pages, proposals, and decision reports for other teams                  | `claude` | `fable`       | omit            |
| Review of `fable` external writing                                                                                                                       | `claude` | `opus`        | `high`          |

Always pass `--effort medium` for Codex: its default differs between machines and CLI versions. If you cannot decide between two rows, choose the cheaper one. Use `xhigh`, `max`, or `ultra` only when the user asks.

A scoped re-review uses `sonnet`, or `opus` when the implementer used `opus` or `fable` or a Codex review found the issue. It never uses Codex.

## Fable

`fable` costs 2.5 times as much as `opus` per token. Spend it on prose, not on reading. A `fable` writer works only from a draft or brief that a `sonnet` or `opus` worker prepared, and does no research: if a fact is missing, it reports the gap and stops. Other than external writing, `fable` is only a retry step.

## Codex

Codex usage limits are much lower than Claude limits, so use Codex only for the two Codex rows.

- Tell `gpt-6.1-sol` that its job is to find what is wrong, not to confirm what is right.
- **Astra budget:** at most 3 `gpt-6-astra` reviews in any 5-hour window. Write each start time to the ledger and count before each new one.
- **Fallback:** if the budget is used up or Codex reports a usage or rate limit, use `opus` with `--effort high` and record it as a ruling. Do not wait for the limit to reset unless the user asks.
- A Codex worker cannot load Claude skills, so its spec names every input, including `reporting.md`, as a file path.

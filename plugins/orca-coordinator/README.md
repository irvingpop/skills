# orca-coordinator

Runs a task as a coordinator session in Orca. The coordinator plans the work, writes a spec for each task, dispatches Orca workers, sends each change to a separate reviewer, and makes the rulings. It does almost no work itself, so its context lasts for long tasks.

## Why use it

A single agent session on a large task fills its context with file reads and test output, reviews its own work, and stops to ask you about every small choice. The coordinator changes that:

- **Long tasks finish in one session.** The coordinator keeps only decisions in its context. Workers and subagents do the reading, editing, and test runs. A ledger file and a handoff file let a new session continue where the last one stopped, also after a context compaction.
- **Every change gets an independent review.** No change is accepted on the word of the agent that made it. A new worker reviews each change against its spec, and the final result gets an adversarial review by a different model (Codex), which catches errors that the same model tends to miss.
- **You are interrupted only for real decisions.** The coordinator asks you before anything hard to reverse, before it changes the goal, and when every option is a guess. It decides everything else, keeps going, and lists each decision with its cost if wrong, so you can check them at the end instead of answering them one by one.
- **Cost matches the work.** Each kind of work goes to the cheapest model that does it well: `haiku` for mechanical sweeps, `sonnet` for normal implementation, `opus` for hard problems, Codex for reviews. The table in `models.md` controls this.
- **Reports you can read cold.** Messages to you say what a thing is, not a label or ID, so you can follow along without reading the ledger or worker output.

```
            you
             |  /coordinator <task>
             v
   +---------------------+      ledger, specs, reports
   |  coordinator        |----> .coordinator/<task>/
   |  plan, rule, report |
   +---------------------+
      |             |
      v             v
  subagents      Orca workers
  (read, search, (authored changes, reviews,
   run tests)     Codex reviews)
```

## Prerequisites

- Claude Code, running inside the Orca app.
- The `orchestration` skill, which Orca installs. The coordinator uses it for all Orca commands.
- Optional: Codex in Orca, for the adversarial review steps. Without it, the coordinator falls back to Claude `opus`.

## Install

```
/plugin install orca-coordinator --marketplace irvingpop/skills
```

On Claude Code older than 2.1.275, run `/plugin marketplace add irvingpop/skills`, then `/plugin install orca-coordinator@irvingpop`.

## Use

```
/coordinator <task to coordinate>
/coordinator resume <handoff file>
```

If another skill also uses the name `coordinator`, run `/orca-coordinator:coordinator` instead.

The coordinator asks you before anything hard to reverse (publish, delete, merge, spend money), before it changes the goal, and when every option is a guess. It decides everything else and lists its rulings in the final report.

## Files

| File | Contents |
|---|---|
| `skills/coordinator/SKILL.md` | Coordinator role, routing, review loop, rulings, handoff, finish report |
| `skills/coordinator/models.md` | Which agent, model, and effort to use for each kind of work. **Edit this for your own plans and limits.** |
| `skills/coordinator/reporting.md` | The report format that workers follow |

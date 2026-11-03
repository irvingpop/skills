# irvingpop/skills

Public Claude Code plugin marketplace (`name: irvingpop`). Each plugin in `plugins/` holds one or more skills in the Agent Skills format.

## Layout

```
.claude-plugin/marketplace.json     catalog: one entry per plugin
plugins/<plugin>/
  .claude-plugin/plugin.json        manifest; "name" must equal the catalog entry "name"
  README.md                         prerequisites, install, use
  skills/<skill>/SKILL.md           frontmatter "name" must equal <skill>
```

## Rules

- Public repo: no employer-internal content, no personal names or paths, no secrets.
- Write skill text for any user ("the user"), not for the author.
- Refer to a skill's own files with `${CLAUDE_SKILL_DIR}/...`, never an absolute or `~` path.
- `SKILL.md` under 500 lines; detail goes in files that `SKILL.md` links to directly.
- Description: third person, says what the skill does and when to use it, no angle brackets.
- Never rename a published plugin. Use `displayName` for a label change, or add to `renames` in `marketplace.json`.
- Plugins have no `version` field on purpose: users track commits on `main`. Ignore the validator warning about it.

## Writing skill text

Write the least instruction that gets the right behavior. The model that reads a skill is capable: give it the goal, the constraints it cannot work out, and the formats that other parties depend on, then trust its judgment for the rest. Redoing work is usually cheap, and an over-specified rule fails in the cases its author did not foresee.

Keep:
- What the model cannot work out: the user's limits (what needs approval, budgets), choices the user made (models, tools, file locations), and contracts with other agents or tools (formats, paths, commands).
- A rule that a test showed is needed. Say what failure it prevents.

Cut:
- Anything another skill or the tool itself already says. Point to it.
- A rule stated twice in one skill.
- Example lists, decision trees, and step-by-step procedures where one sentence of principle does the job.
- Rules that a tool's own error messages already enforce.

Before you add a rule, ask: would a capable colleague get this wrong without it? If not, leave it out.

## Adding a plugin

1. Create `plugins/<plugin>/` with `plugin.json`, `README.md`, and `skills/`.
2. Add the catalog entry to `marketplace.json`.
3. Add a row to the plugin table in `README.md`.
4. Validate.

## Validate

```
claude plugin validate .
```

Must pass (warnings for `version` and the plugin icon are expected). CI runs the same command.

## Test

This checkout is registered as the `irvingpop` marketplace on the author's machine, so edits load in place: run `/reload-plugins` or start a new session. No version bump or update is needed.

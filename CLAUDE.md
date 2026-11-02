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

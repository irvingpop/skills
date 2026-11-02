# irvingpop/skills

Agent skills by Irving Popovetsky, packaged as a Claude Code plugin marketplace. The skills follow the [Agent Skills](https://agentskills.io) format.

## Install (Claude Code)

```
/plugin marketplace add irvingpop/skills
/plugin install <plugin>@irvingpop
```

To get updates automatically, open `/plugin`, go to **Marketplaces**, select `irvingpop`, and select **Enable auto-update**. Otherwise run `/plugin marketplace update irvingpop`.

If you have no GitHub SSH key, set `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` before you add the marketplace.

## Plugins

| Plugin | Skills | What it does |
|---|---|---|
| [`orca-coordinator`](plugins/orca-coordinator) | `coordinator` | Runs a task as an Orca coordinator session that plans, dispatches workers, reviews, and makes rulings. Requires Orca. |

## Layout

```
.claude-plugin/marketplace.json        marketplace catalog (name: irvingpop)
plugins/<plugin>/
  .claude-plugin/plugin.json           plugin manifest; name matches the catalog entry
  README.md                            prerequisites and usage
  skills/<skill>/SKILL.md              one directory per skill
```

## Develop

```
git clone git@github.com:irvingpop/skills.git
claude plugin marketplace add ./skills    # loads plugins from your checkout; /reload-plugins picks up edits
claude plugin validate ./skills
```

Rules for this repo:

- Skill `name` matches its directory, in lowercase with hyphens. The description says what the skill does and when to use it.
- Keep `SKILL.md` under 500 lines. Put detail in files that `SKILL.md` links to directly.
- Refer to the skill's own files with `${CLAUDE_SKILL_DIR}`, never with an absolute path.
- Never rename a published plugin. To change only its label, use `displayName`. To rename or remove one, add an entry to `renames` in `marketplace.json`.
- Plugins have no `version` field, so users get each commit on `main`.

## License

[Apache-2.0](LICENSE)

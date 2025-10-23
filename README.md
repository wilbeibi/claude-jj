# Claude Code + Jujutsu

Hooks and a small Claude skill that give each Claude Code task its own Jujutsu change.

## Setup

1. Install [Jujutsu](https://github.com/martinvonz/jj) and [Claude Code](https://claude.ai/code). Work in a jj-backed repo.
2. Copy `.claude/settings.json` into `~/.claude/settings.json` (merge if you already have settings).
3. Copy `skills/jj-auto/SKILL.md` to `/mnt/skills/user/jj-auto/SKILL.md`.

Claude Code will now spin up a fresh jj change whenever a tool run starts.

## Workflow

Run Claude Code as usual:

```bash
claude "add user authentication"
```

The hooks:
1. Create a new change before the first file edit.
2. Show a summary when the task ends.
3. Leave the change active so you can review or edit it.

Common follow-ups:

```bash
jj diff        # Inspect the change
jj commit      # Accept it
jj abandon @   # Drop it
jj split       # Keep only part of it
```

Subsequent prompts that refine the same task reuse the current change; switching topics creates a new one.

## Notes

- Jujutsu keeps change IDs stable through rebases, so Claude’s changes stack cleanly.
- No staging area is involved—jj tracks the working copy directly.
- Use `jj op log` / `jj op restore` if you ever need to undo a hook run.

## Troubleshooting

- Nothing happens? Confirm the skill path and hook config, and check `jj status` to ensure you are inside a jj repo.
- Change missing? Look at `jj log` or `jj op log` to confirm where you are.
- Need to recover? `jj undo` reverts the last operation; `jj abandon @` drops the active change.

## License

MIT — Issues and PRs welcome.

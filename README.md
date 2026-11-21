# assign

A Claude Code skill that splits a big task into independent pieces and hands each piece to its own session or subagent. Each piece gets a model and reasoning effort matched to how hard it is: cheap, fast models take easy pieces, stronger models take hard ones.

The session you run it from acts as the manager. It reads the task, decides who does what, writes short briefs and reads short reports. It does not do the work itself.

## Use

Type `/assign` followed by your task, or say things like "assign this", "split this up" or "farm this out".

Sessions are started with `start_session`, which is available in the Claude desktop app's Code tab. In other setups the skill still splits work across subagents.

## Install

In a terminal session of Claude Code:

```
/plugin install assign --marketplace minhphamdxp3-dev/assign-skill
```

Answer `y` to add the marketplace, then pick the user scope.

From a regular terminal:

```
claude plugin install assign --marketplace minhphamdxp3-dev/assign-skill
```

In the Claude desktop app: click **+** next to the prompt box, then **Plugins**, **Add marketplace** with `minhphamdxp3-dev/assign-skill`, then **Add plugin**.

Prefer a single file? Copy `skills/assign/SKILL.md` into `~/.claude/skills/assign/SKILL.md`.

## License

MIT

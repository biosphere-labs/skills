# Skills

Agent skills from Biosphere Labs. Each folder holds one skill as a `SKILL.md` file, the format Claude Code and other agents load.

| Skill | What it does |
|---|---|
| [skill-scribe](skill-scribe/SKILL.md) | Starts a scribe subagent that watches you do a process live, then turns its notes, checked against the session's traces, into a reusable skill for that process. Optionally repeats the process across several instances with a state machine. |

## Install

Claude Code: copy the skill's folder into `~/.claude/skills/` (for all projects) or `.claude/skills/` in a project.

```bash
git clone https://github.com/biosphere-labs/skills.git
cp -r skills/skill-scribe ~/.claude/skills/
```

Other agents that read `SKILL.md` skills: copy the folder to wherever they load skills from.

## Use

For skill-scribe, say something like this before you start the process you want captured:

> Set up a scribe to watch me and write down everything I do until I tell you to stop.

Then do the work, and say stop when you're done.

## Licence

MIT. See [LICENSE](LICENSE).

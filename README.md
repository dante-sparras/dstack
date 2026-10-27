# dstack

My personal collection of agent skills, installable with the [`skills`](https://github.com/vercel-labs/skills) CLI.

## Install

```sh
# Interactive: pick skills and agents
npx skills add dante-sparras/dstack

# One skill
npx skills add dante-sparras/dstack --skill hello-dstack

# Everything, globally, for Claude Code, no prompts
npx skills add dante-sparras/dstack -g -a claude-code -y

# List what's available without installing
npx skills add dante-sparras/dstack --list

# Pull in updates
npx skills update
```

`bunx skills ...` works the same way.

## Skills

| Skill | Description |
| --- | --- |
| [hello-dstack](skills/hello-dstack/SKILL.md) | Smoke test that confirms dstack skills are installed. |

## Adding a skill

1. `cd skills && npx skills init <name>`
2. Edit `skills/<name>/SKILL.md`. The frontmatter needs a lowercase, hyphenated `name` and a `description` that says what the skill does and when to use it.
3. Test locally from the repo root: `npx skills add ./ --list`
4. Add it to the table above, then commit and push.

# claude-skills

A collection of [Claude Code](https://claude.com/claude-code) skills. Each skill lives in its own folder with a `SKILL.md` (the instructions Claude follows) and a `README.md` (how to install and use it).

## Skills

| Skill | What it does |
| --- | --- |
| [qa-ticket](qa-ticket/) | Prepares you to manually QA a Jira ticket on staging: exact test steps, gaps the fix might miss, and a ready-to-paste PASS/FAIL Jira comment. Read-only, never touches staging. |
| [repo-intro](repo-intro/) | First-look orientation for an unfamiliar repo: what it is, file structure, tech stack, and architecture, in a fixed order. Read-only. |

## Install

Copy a skill's `SKILL.md` into your Claude Code skills directory, keeping the folder name:

```sh
mkdir -p ~/.claude/skills/repo-intro
cp repo-intro/SKILL.md ~/.claude/skills/repo-intro/SKILL.md
```

To install every skill at once:

```sh
for d in */; do
  [ -f "$d/SKILL.md" ] && mkdir -p ~/.claude/skills/"$d" && cp "$d/SKILL.md" ~/.claude/skills/"$d"
done
```

Then run it in Claude Code with `/<skill-name>`, e.g. `/repo-intro`. See each skill's README for arguments and requirements.

## Adding a skill

1. Create a folder named after the skill.
2. Add a `SKILL.md` with `name` and `description` frontmatter, followed by the instructions.
3. Add a `README.md` covering what it does, install, and use.
4. Add a row to the table above.

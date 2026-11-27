# claude-skills

A collection of Claude Code skills. Each skill lives in its own folder with a `SKILL.md` and a `README.md`.

## Skills

| Skill | What it does |
| --- | --- |
| [fantasy-matchup](fantasy-matchup/) | Weekly Yahoo fantasy basketball (9-cat H2H) matchup report: category locks, toss-ups, both teams' strengths and weaknesses, and waiver categories to target. Writes an HTML file and opens it. |
| [jira-sheets-test-cases](jira-sheets-test-cases/) | Finds Jira tickets in Ready for QA with no tab in your test-case Google Sheet, and writes one tab per ticket with one test case per acceptance criterion. Read-only on Jira; only adds new tabs. |
| [qa-ticket](qa-ticket/) | Prepares you to manually QA a Jira ticket on staging: exact test steps, gaps the fix might miss, and a ready-to-paste PASS/FAIL Jira comment. Read-only, never touches staging. |
| [repo-intro](repo-intro/) | First-look orientation for an unfamiliar repo: what it is, file structure, tech stack, and architecture, in a fixed order. Read-only. |

## Install

Copy a skill's `SKILL.md` into your Claude Code skills directory, keeping the folder name:

```sh
mkdir -p ~/.claude/skills/repo-intro
cp repo-intro/SKILL.md ~/.claude/skills/repo-intro/SKILL.md
```
Then run it in Claude Code with `/<skill-name>`, e.g. `/repo-intro`. See each skill's README for arguments and requirements.

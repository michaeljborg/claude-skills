# repo-intro

A Claude Code skill for your first prompt in an unfamiliar repo. Claude explores the codebase read-only and gives a short overview in a fixed order:

1. **What it is**: a few lines on what the project does
2. **Structure**: the file tree, with a table of sub-apps for monorepos
3. **Tech stack**: a short bullet list
4. **Architecture**: how the pieces connect
5. **Worth knowing**: gotchas, setup quirks, or your focus area. This section only appears when there's something worth flagging.

Claude never edits files, installs dependencies, or runs builds.

## Install

Copy `SKILL.md` to `~/.claude/skills/repo-intro/SKILL.md`.

## Use

```
/repo-intro
```

To go deeper on one area, add a focus. You still get sections 1–4 in short form:

```
/repo-intro auth
/repo-intro apps/api
```

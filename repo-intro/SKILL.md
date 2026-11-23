---
name: repo-intro
description: First-look orientation for a repo the user has just opened. Use when the user runs /repo-intro, or asks for an overview, intro, or rundown of an unfamiliar codebase ("what is this repo", "walk me through this project", "get me oriented"). Claude explores the repo read-only and reports, in a fixed order, what it is, its file structure, its tech stack, and its architecture, then adds a short notes section only if something non-obvious is worth flagging. Never edits files or runs builds.
---

# Repo intro

The user has opened this repo for the first time and wants to get oriented fast. Give a short, accurate overview in a fixed order. Every claim should come from files you actually read. They can ask follow-ups for depth, so keep this brief.

## Rules

- **Read-only.** Don't edit, install, build, run tests, or start servers. Reading files, `ls`, `find`, `git log`, and `git remote -v` are fine.
- **Grounded.** Base claims on files you've read. If you're inferring something (e.g. "looks like a deploy target, based on `fly.toml`"), say so. Don't pad with generic descriptions of well-known frameworks.
- **Brief.** Use the length limits below. A short answer the user can scan in under a minute is better than a complete one.
- **Respect focus.** If the user passes an argument (a subdirectory, or a topic like "auth" or "the API"), still give sections 1–4 in short form, then give that topic the depth in section 5.

## Explore (before writing anything)

Do this in roughly this order and stop when you have enough. Batch independent reads in parallel.

1. **Docs and agent files:** `README*`, `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING*`, `docs/` index, `ARCHITECTURE*`. Treat these as claims to check against the code, since they're often stale.
2. **Layout:** top-level listing plus 2 levels deep, ignoring `node_modules`, `.git`, `dist`, `build`, `vendor`, `.venv`, `target`, and other generated or vendored folders. `git ls-files | head`, or counting files per top-level dir, helps size things.
3. **Manifests:** `package.json` (+ workspaces), `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `pyproject.toml`, `requirements*.txt`, `go.mod`, `Cargo.toml`, `Gemfile`, `pom.xml`/`build.gradle`, `composer.json`, `*.csproj`, `pubspec.yaml`, etc. Read their scripts/commands too.
4. **Infra and config:** `Dockerfile`, `docker-compose*`, `.github/workflows/`, `terraform/`, `k8s/`, `serverless.yml`, `vercel.json`, `fly.toml`, `.env.example`, DB migrations and schema files.
5. **Entry points:** main server/app files, route definitions, CLI entry, `index`/`main`/`app` files of each sub-app. Skim them to confirm how the pieces connect.
6. **History (light):** `git log --oneline -15` and `git log -1 --format=%cr` for activity level and current focus. Skip if there's no git.

**Monorepos/large repos:** identify each sub-app or package first, then do steps 3–5 per unit. If there are many units (roughly 6+), you can fan out with Explore subagents, one per group. Each subagent returns a 2–3 line summary of its unit: purpose, stack, and entry point.

## Output

Use exactly these headers, in this order. No preamble before section 1.

### 1. What it is
2–4 lines. What the project is, what it does, and who or what uses it (end users, internal team, other services, a library consumed by X). Include the repo/product name.

### 2. Structure
A tree of the meaningful directories only, each with a short `# comment`. Collapse boring or generated parts. Keep it to about 25 lines.

If there are multiple sub-apps/packages/services, lead with a table instead (or as well), then a compact tree:

| Unit | Path | What it does | Stack |
|---|---|---|---|

Call out where the user is most likely to work (e.g. "most product code lives in `apps/web/src/features/`").

### 3. Tech stack
A tight bullet list, about 4–8 bullets, grouped as relevant: language/runtime, frameworks, data (DB, ORM, cache, queues), infra/deploy, tooling (package manager, test, lint, CI). Include versions only when they matter (e.g. a major version that changes how you write code, like React 19 or Python 2).

### 4. Architecture
3–6 bullets, or one small ASCII diagram plus 2–3 bullets, describing how the pieces talk to each other: request/data flow, main boundaries (client ↔ API ↔ DB, services ↔ queue), external integrations, and where state lives. Name the actual files/dirs that are the key seams.

### 5. Worth knowing *(only if needed)*
Include this section **only** if something non-obvious would trip up a newcomer or the user asked for focus. If nothing qualifies, leave the section out entirely, with no "nothing notable" placeholder. Max ~6 bullets. Good candidates:
- How to run it locally, if it's not obvious (unusual setup, required services, seed scripts, env vars from `.env.example`).
- Generated code or "don't edit" areas (codegen, OpenAPI clients, migrations).
- Conventions from CLAUDE.md/CONTRIBUTING that affect how to make changes.
- Mismatches between docs and code, dead/legacy dirs, half-finished migrations.
- Repo health signals: no tests, no CI, last commit long ago, many TODO/FIXME in core paths.
- The topic the user asked to focus on.

End with one line offering likely next dives, based on what you found (e.g. "Want me to trace the checkout flow or explain the job queue?"). Don't use a generic offer.

## Edge cases

- **Empty or near-empty repo:** say so in one or two lines and list what's there. Skip sections 2–4 if they'd be empty.
- **Not a code repo** (docs, dotfiles, config, data): adapt the sections. Keep 1 and 2, and replace 3–4 with a short "How it's organized/used" section.
- **No README:** infer the purpose from code and manifests, and say it's inferred.
- **Huge repo:** explore at the unit level only. Don't attempt exhaustive reading; say which areas you skimmed versus read.

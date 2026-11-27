# jira-sheets-test-cases

A Claude Code skill that keeps a Google Sheet of test cases in step with a Jira board. Claude finds every ticket in **Ready for QA** that has no tab yet, duplicates the template tab for each one, links the ticket, and writes one test case per acceptance criterion (desktop only, short steps, result dropdowns left blank). It never edits Jira, never touches existing tabs or tester results, and never opens the QA site.

## Install

Copy `SKILL.md` to `~/.claude/skills/jira-sheets-test-cases/SKILL.md`, then fill in the placeholders under "Fixed references":

- `<your-site>`: your Atlassian site name
- `<PROJECT_KEY>`: the Jira project key
- `<SPREADSHEET_ID>`: the ID from the sheet's URL
- `<TEMPLATE_TAB_NAME>` / `<TEMPLATE_SHEET_ID>`: the tab to copy for each ticket, and its `gid` from the URL

The template tab should have the link in row 1, headers in row 2 (`Test case | Preconditions | Test Steps | Expected result | Web | IOS | Android | Notes`), and result dropdowns in E–G. Optionally add your own project's terms in the "Project-specific terms" section.

## Use

```
/jira-sheets-test-cases
```

Claude replies with how many tabs it created and which tickets, plus a few notes only if something needs the tester's attention (skipped mobile-only ACs, copy TBC, contradictions).

Needs the Atlassian and Google Sheets connectors (authenticate with `/mcp`).

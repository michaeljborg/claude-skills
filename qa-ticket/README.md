# qa-ticket

A Claude Code skill for manually QA-ing a Jira ticket on staging. Claude reads the ticket and finds the related PR. It then gives you exact steps to test the change, points out where the fix might not cover the real problem, and drafts a PASS Jira comment for you to paste. If you report a problem, it switches the comment to FAIL. Claude never edits code and never touches staging.

## Install

Copy `SKILL.md` to `~/.claude/skills/qa-ticket/SKILL.md`.

## Use

```
/qa-ticket PROJ-123
```

You can also paste a Jira URL. Run your test using Claude's steps. If something fails, tell Claude and it will rewrite the comment as FAIL.

Works best with the Atlassian connector (authenticate with `/mcp`) and `gh` for PR lookups. Without the connector, paste the ticket text instead.

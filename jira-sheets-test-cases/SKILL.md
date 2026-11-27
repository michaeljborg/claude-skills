---
name: jira-sheets-test-cases
description: Find Jira tickets in "Ready for QA" that have no tab yet in the team's test-case Google Sheet, then write one tab per ticket with one test case per acceptance criterion, following the team's sheet rules. Use when the user runs /jira-sheets-test-cases or asks to check for new Ready for QA tickets, write the missing test cases, or catch the test sheet up with Jira.
---

# Jira test cases in Google Sheets

Keep the test-case sheet in step with the Jira board: every ticket in **Ready for QA** gets its own tab of test cases. Only add what's missing; never rewrite or remove existing tabs.

## Fixed references

Fill these in before the first run.

| What | Value |
|---|---|
| Jira site / project | cloudId `<your-site>.atlassian.net`, project `<PROJECT_KEY>` |
| Spreadsheet | ID `<SPREADSHEET_ID>` |
| Template tab | `<TEMPLATE_TAB_NAME>`, sheetId `<TEMPLATE_SHEET_ID>` |
| Ticket link | `https://<your-site>.atlassian.net/browse/<PROJECT_KEY>-<n>` |

Template layout (re-read it each run; the user may change it):
- Row 1: A1 `JIRA User Story Link:`, with the link in B1.
- Row 2: headers `Test case | Preconditions | Test Steps | Expected result | Web | IOS | Android | Notes` (A–H).
- Rows 1–2 frozen.
- E3:G100 hold dropdowns: Passed, Failed, To Do, Blocked, n/a.

## Hard rules

- **Jira is read-only.** Never comment, edit or transition a ticket.
- **Touch only the tabs you create.** Never edit the template tab, a regression tab or an existing ticket tab. Above all, never clear or overwrite the tester's results in E–G.
- **Don't open the QA site** and don't run the app. Cases come from the ticket.
- Read the `anthropic-skills:google-workspace` skill and its Sheets reference before the first sheet write in a session.

## Steps

### 1. Load the tools
Load them with ToolSearch:
`select:mcp__claude_ai_Atlassian_MCP__searchJiraIssuesUsingJql,mcp__claude_ai_Atlassian_MCP__getJiraIssue,mcp__claude_ai_Google_Sheets__get_spreadsheet,mcp__claude_ai_Google_Sheets__get_values,mcp__claude_ai_Google_Sheets__update_values,mcp__claude_ai_Google_Sheets__update_spreadsheet`

If a connector is missing or fails with "needs to be connected", stop and ask the user to reconnect it with `/mcp`.

### 2. Find the missing tickets
1. Search Jira with `project = <PROJECT_KEY> AND status = "Ready for QA" ORDER BY key ASC`, fields `summary`, `issuetype`, `parent` and the Sprint field (often `customfield_10020`; IDs vary per site). The result can be large; if it's saved to a file, use jq.
2. Call `get_spreadsheet` with `fields: ["sheets.properties.sheetId","sheets.properties.title","sheets.properties.index"]`.
3. Missing = the Ready for QA keys with no tab titled `<PROJECT_KEY>-<n>`.
4. Go ahead without listing them. Only if more than 8 are missing, ask first whether to do all of them or only the active sprint.

If nothing is missing, give the empty report (see Report) and stop.

### 3. For each missing ticket, in key order
1. **Read the ticket:** `getJiraIssue` with `view: "evidence"`.
   - Read the description and **all comments**. A comment marked as a requirement change replaces the description's ACs; use the latest.
   - Note links to other tickets ("ships with", "blocked by").
2. **Create the tab:** `update_spreadsheet` with `duplicateSheet` from sourceSheetId `<TEMPLATE_SHEET_ID>`, `newSheetName: "<PROJECT_KEY>-<n>"`. Record the new sheetId.
3. **Move it into place:** tab order is any regression tab, the template tab, then ticket tabs in ascending number. Use `updateSheetProperties` with `fields: "index"`.
4. **Add the link** to B1 with `update_values`: `=HYPERLINK("https://<your-site>.atlassian.net/browse/<PROJECT_KEY>-<n>", "<PROJECT_KEY>-<n>: <summary>")`.
5. **Write the cases** to `'<PROJECT_KEY>-<n>'!A3:H<2+count>`, following the case rules below.
6. **Format the case rows:** `repeatCell` on rows 3..2+count, columns A–H, with `{"wrapStrategy":"WRAP","verticalAlignment":"TOP"}` and fields `userEnteredFormat.wrapStrategy,userEnteredFormat.verticalAlignment`.
7. **Check the dropdowns:** read `dataValidation` on E3:G3. If it's missing, copy it from the template with `copyPaste` `pasteType: "PASTE_DATA_VALIDATION"`:
   - source `{sheetId:<TEMPLATE_SHEET_ID>,startRowIndex:2,endRowIndex:100,startColumnIndex:4,endColumnIndex:7}`
   - destination: the same range on the new tab
8. **Verify** by reading `'<PROJECT_KEY>-<n>'!A1:H<2+count>` back:
   - B1 is the link.
   - The case count equals the AC count.
   - None of these words appear: phone, tablet, mobile, iOS, Android, swipe, 375, 768.

## Case rules

- **One test case per acceptance criterion, no more and no fewer.**
  - Don't split an AC across cases, and don't add visual-check, edge-case or comment-derived extras.
  - Write column A as `TCn – <short title> (ACn)` with an en dash, where n is the AC number.
- **Unnumbered criteria or a plain bullet list:** one case per bullet, numbered AC1… in order.
- **No ACs at all:** at most 3 cases, one per stated requirement, without `(ACn)`.
- **Desktop only for now.**
  - Leave out phone, tablet, swipe and small widths.
  - If an AC is entirely about phone or tablet, skip it, keep TC numbering matched to AC numbers, and tell the user.
  - If an AC mixes desktop and mobile, keep only the desktop part.
- **Short and plain.**
  - B Preconditions: 1–2 `• ` bullets.
  - C Test Steps: at most 4 numbered steps (`1. …\n2. …`), with exact page names and button labels.
  - D Expected result: at most 3 `• ` bullets, taken from the AC wording.
- **E / F / G** (Web, IOS, Android): always `""` on a new tab.
- **H Notes:** only what the tester must know, otherwise `""`. For example:
  - `Needs CMS` (the case changes CMS content)
  - `Needs dev` (sample/fixture content, failure states, infra)
  - `Ships with <PROJECT_KEY>-<n>`
  - `Copy TBC` / `Design TBC`
- **Never start a cell with `=`, `+` or `-`** (except the B1 formula); the API parses input like the UI. Use `•` for bullets.

### Project-specific terms (optional)

Add notes here on how your project's terms translate into test cases. For example:
- **CMS limits:** actions testers can't do in your CMS, and the workaround to use instead.
- **Server-side failures** that can't be triggered from the browser: mark them `Needs dev`.
- **Hard-coded or fixture-only sections** that don't read the CMS: changing their content is `Needs dev`, not `Needs CMS`.

## Report

Start the final reply with these two lines:

```
New test cases found: <number of new tabs created>
Test cases: <KEY-n, KEY-n, KEY-n>
```

- List the ticket keys of the tabs created, in key order, separated by commas.
- If nothing was missing: `New test cases found: 0` and `Test cases: none`.

Then, **only if there's something the tester needs to know**, add a very short report of at most 5 bullets, one line each, under `Notes:`. For example:
- `Failed: KEY-n (reason)`
- `KEY-n: AC4 skipped (mobile-only)`
- `KEY-n: AC2 blocked, copy TBC`
- `KEY-n: AC3 contradicts KEY-m`
- `KEY-n: dev says not deployed yet`

If there's nothing to flag, end after the two lines. No tables, summaries or restated rules.

For more than about 5 missing tickets, split them across up to 4 parallel general-purpose agents, ~5 tickets each. Give each this skill's steps and rules, and the exact list of keys it owns.

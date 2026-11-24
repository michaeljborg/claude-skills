---
name: fantasy-matchup
description: Weekly matchup analysis for Yahoo fantasy basketball, standard 9-cat head-to-head (FG%, FT%, 3PTM, PTS, REB, AST, ST, BLK, TO). Use when the user runs /fantasy-matchup or pastes a matchup ("My Team vs Opponent") plus a league stats grid and wants to know which categories they'll win, which are toss-ups, and what to target on waivers. Claude compares the two teams against each other and the league, writes a styled HTML report to disk, and opens it in the browser.
---

# Fantasy basketball matchup analysis

Standard Yahoo **9-cat head-to-head**. Each week you win or lose each category separately; the team that wins 5+ of the 9 wins the week.

## Inputs

The user pastes two things:

1. **The matchup:** `Team 1 vs Team 2`. Team 1 is always the user's team; Team 2 is this week's opponent.
2. **The league stats grid:** one row per team, tab- or space-separated, with columns `Team Name, FG%, FT%, 3PTM, PTS, REB, AST, ST, BLK, TO`. These are **season totals** (counting stats) and **season averages** (FG%, FT%) X weeks into the season.

Optional extras, used if present: the number of weeks played, the week number, and any current-week live stats from the Yahoo matchup page.

Parsing:
- Team names contain spaces, so split from the right: the last 9 fields are stats and everything before them is the name.
- Match the matchup names to grid rows case-insensitively and ignore punctuation. If a name doesn't clearly match exactly one row, ask the user which team they meant. Don't guess.
- If a column is missing or a row won't parse, say which one and stop.

## Rules

- **TO is the reverse category: lower wins.** Apply this everywhere: comparisons, ranks, colors, and labels.
- **Compare counting stats as rates, not raw totals.** Totals reflect games played as well as talent. A team with far lower totals across every counting category (e.g. ~30% fewer PTS, REB, and AST than the league) has probably played fewer games, from injuries, empty slots, or not streaming. Flag that in the report, because a full week of games closes the gap. If weeks played is given, show per-week numbers. Either way, compare the two teams with the % difference of their totals, since they've played the same number of weeks.
- **Percentages are rate stats.** Compare them by the absolute gap (e.g. .487 vs .476 = +.011). Don't treat them as % differences.
- **Do the math with a script.** Write a short Python script to the scratchpad, pass it the pasted grid, and use its numbers. Don't do arithmetic in your head. Show the numbers in the report.
- **Be honest about uncertainty.** These are season totals. One week of a 9-cat matchup swings a lot on games played, injuries, and streaming. Call them projections, not guarantees.

## Analysis

### 1. Per-category comparison
For each of the 9 categories, compute:
- Team 1 value, Team 2 value, and the edge (who leads and by how much)
- Each team's **league rank** (1 = best; for TO, 1 = fewest)
- A **verdict** using these thresholds:

| Verdict | Counting stats (% gap) | FG% (abs gap) | FT% (abs gap) |
| --- | --- | --- | --- |
| **Lock** | ≥ 15% | ≥ .015 | ≥ .030 |
| **Lean** | 6–15% | .006–.015 | .012–.030 |
| **Toss-up** | < 6% | < .006 | < .012 |

Compute the % gap as `|a − b| / avg(a, b)`.

Then adjust with judgment, and say why whenever you do:
- Low volume is volatile. ST and BLK swing more week to week than PTS or REB, so downgrade a borderline Lock to Lean in those categories.
- FT% is volatile for teams with few attempts. FG% is more stable.
- If one team looks short on games played (see Rules), counting-stat edges in its favor are stronger than they look, and edges against it are weaker.

### 2. Team profiles
For **each** team:
- **Strengths:** categories ranked top 3 in the league, or clear edges over the opponent.
- **Weaknesses:** categories ranked bottom 3, or clear deficits.
- **Build read:** one line on the team's archetype, e.g. "big-man build: BLK/REB/FG%, weak FT%" or "guard-heavy: 3PTM/AST/FT%, high TO". Say if it looks like a team is punting a category.

### 3. Projected result
- **Locks** for each team: the categories each should win comfortably.
- **Toss-ups:** the categories that decide the week. For each, give one line on what tips it.
- **Projected score**, e.g. `5–3–1` (W–L–T). Count each toss-up as a tie, then give a best case (win every toss-up) and a worst case.

### 4. Waiver targets
Recommend which categories the user should chase on waivers this week. Prioritize in this order:
1. **Toss-ups and slight Leans against the user.** A small boost flips these.
2. **Slight Leans in the user's favor** that the opponent could steal.
3. **Never chase a category the user is badly losing.** Say plainly that it's a punt for the week and that chasing it wastes moves.

For each target category, give:
- **Why:** the gap, and what flips it (e.g. "~4 extra BLK flips it").
- **Player archetype:** e.g. "backup rim-protecting C", "volume 3&D wing", "high-FT% guard", "low-usage big for FG% and low TO".
- **Watch out for:** side effects on the other categories. Volume scorers add TO. Bigs can hurt FT%. Streaming for counting stats raises TO and can drag the percentages down.

You don't have player data, so recommend archetypes, not named players. If the user asks for names, say you'd need their league's waiver list or current player stats.

Then add a one-line **streaming note**: if games played looks like the deciding factor, say whether streaming for more games helps or hurts the user given which categories are toss-ups. Streaming helps counting stats and hurts TO.

## Output

Write a **single self-contained HTML file** with inline CSS, no external requests, and support for both light and dark mode. Save it to:

```
./fantasy-matchups/<YYYY-MM-DD>-<team1-slug>-vs-<team2-slug>.html
```

The path is relative to the current directory. Create the folder if it doesn't exist. Then open the file with `open <path>` on macOS, `xdg-open` on Linux, or `start` on Windows.

Page layout, top to bottom:

1. **Header:** `Team 1 vs Team 2`, the date, and the projected score as a large headline (e.g. `Projected 5–3–1`), with the best and worst case underneath.
2. **Category scoreboard:** one row per category, in the order FG%, FT%, 3PTM, PTS, REB, AST, ST, BLK, TO. Columns: Category, Team 1 value (league rank), Team 2 value (league rank), Edge, Verdict. Shade each row by verdict and side: green for a Team 1 Lock or Lean (darker for Lock), red for a Team 2 Lock or Lean, amber for a Toss-up. Mark TO with "(lower wins)".
3. **Team profiles:** two cards side by side (stacked on narrow screens), each with Strengths, Weaknesses, and Build read.
4. **Locks and toss-ups:** Team 1 locks, Team 2 locks, and the toss-up list with a one-line note on what tips each.
5. **Waiver targets:** a ranked list, each with Category, Why, Player archetype, and Watch out for. Then the punt note and the streaming note.
6. **Footnote:** "Based on season totals through week X. Projections, not guarantees." Add the games-played caveat here if you flagged one.

Keep the text tight: short phrases and numbers, not paragraphs. Use a system font stack, readable widths, and color only where it carries meaning.

Finish with a short chat reply: the file path, the projected score, the toss-ups, and the top 1–2 waiver targets. Don't repeat the whole report in chat.

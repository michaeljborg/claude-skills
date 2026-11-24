# fantasy-matchup

A Claude Code skill for weekly Yahoo fantasy basketball matchups in **standard 9-cat head-to-head** leagues (FG%, FT%, 3PTM, PTS, REB, AST, ST, BLK, TO). Paste your matchup and the league stats grid, and Claude writes an HTML report and opens it in your browser. The report covers:

1. **Projected score:** a best case, a worst case, and a scoreboard of all 9 categories rated Lock, Lean, or Toss-up
2. **Team profiles:** each team's strengths, weaknesses, and build
3. **Locks vs. toss-ups:** which categories should be safe for each side, and which ones will decide the week
4. **Waiver targets:** which categories to chase, which player archetypes to look for, the side effects to watch, and which categories to punt

Reports are saved to `./fantasy-matchups/<date>-<team1>-vs-<team2>.html`.

## Install

Copy `SKILL.md` to `~/.claude/skills/fantasy-matchup/SKILL.md`.

## Use

```
/fantasy-matchup
The hooligans vs Shot Chucker FC
Week 6

Team Name    FG%    FT%    3PTM    PTS    REB    AST    ST    BLK    TO
The hooligans    .487    .761    258    2346    805    512    134    93    339
Shot Chucker FC    .445    .813    366    2854    1107    509    185    188    339
...
```

Put your team first. Paste the whole league grid, not just the two teams, because league ranks feed into the strengths and weaknesses. The week number is optional but helps.

Notes:
- TO is treated as the reverse category (fewer wins).
- The grid shows season totals, so a team that has played fewer games will look weaker in counting stats than it really is. The report flags this when it spots it.
- Waiver suggestions are player archetypes, not named players, because the skill doesn't have access to your league's waiver wire.

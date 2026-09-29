# Game Notes Agent — Project Brief

This project is an agentic workflow built entirely from Claude Code features (a skill,
three subagents, and project settings). There is no Python or application code. Everything
runs inside Claude Code in VS Code.

## The business function
Every NFL team's Communications (PR) department produces weekly pregame **game notes**:
a packet for broadcasters and reporters with the matchup, records, statistical leaders,
series history, injuries, milestones and storylines, plus social media copy. Staff build
it by hand from many sources every week. This workflow automates the research, drafting
and fact-checking, and keeps a human editor in charge of approval.

## How the workflow runs
Typing `/game-notes BUF` (any team abbreviation) runs the skill in
`.claude/skills/game-notes/SKILL.md`, which orchestrates:
1. **data-scout** subagents (in parallel, one per team) download raw data from ESPN's
   public JSON endpoints into `data/<run>/` and write a facts sheet with a source for every number.
2. **notes-writer** subagent drafts the notes using ONLY the facts sheets.
3. **fact-checker** subagent verifies every claim against the facts sheets and raw files.
   Failed checks go back to the writer (max 2 rounds).
4. **Human approval**: the editor approves or requests changes in the chat.
5. **Publish**: final Markdown + styled HTML in `output/`, and a row in `output/approval_log.csv`.

## Rules for every agent in this project
- Never state a statistic, record, score, date or name that is not in the downloaded data.
- Never use memory or outside knowledge for facts about players, teams or games.
- If data is missing, leave it out and say so in the facts sheet. Do not guess.
- Avoid arithmetic. Prefer numbers the data already provides. If a calculation is
  unavoidable, show it in the facts sheet (e.g. "computed: 101 / 3 = 33.7").
- Keep raw data files unchanged; they are the audit trail.
- Team abbreviations follow ESPN (e.g. BUF, KC, LAR, LAC, LV, SF, WSH).

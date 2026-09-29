# Game Notes Agent (NFL)

An agentic workflow for a real sports business function: the weekly pregame **game notes**
an NFL team's Communications department sends to broadcasters and reporters.

Built entirely with Claude Code features in VS Code: one skill that coordinates three
subagents with deliberately limited tools. There is no application code.

- **data-scout** downloads ESPN data and writes a facts sheet where every number cites its source file
- **notes-writer** drafts the notes with no web access, using only the facts sheets
- **fact-checker** (read-only) verifies every claim; failures go back to the writer
- **A human editor** approves, requests changes, or cancels before anything is published

## Run it
1. Install VS Code and the Claude Code extension, and sign in.
2. Open this folder and trust it.
3. In the Claude Code panel, type `/game-notes BUF` (any NFL team abbreviation).
   Use `/game-notes BUF saved` to rerun from previously downloaded data.

See [BUILD_GUIDE.md](BUILD_GUIDE.md) for the full walkthrough.

Data comes from ESPN's public but unofficial JSON endpoints. Student project; not affiliated
with the NFL, ESPN, or any team.

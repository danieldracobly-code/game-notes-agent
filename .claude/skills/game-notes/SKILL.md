---
name: game-notes
description: Research, write, fact-check and publish NFL pregame game notes for a team's next game, with human approval before publishing.
argument-hint: "<TEAM> [saved]   e.g. /game-notes BUF  or  /game-notes BUF saved"
disable-model-invocation: true
---

# /game-notes workflow

You are the Communications department's workflow coordinator. You delegate the work to
subagents, keep the editor informed, and never publish without the editor's approval.

Arguments: $ARGUMENTS
- The first word is OUR_TEAM (ESPN abbreviation, uppercase it).
- If the word `saved` appears, use SAVED MODE (see Step 1b).

Narrate briefly as you go (one line per step) so an audience watching can follow along.

## Step 0 — Plan
State a 3-5 line plan for this run in the chat.

## Step 1a — Research (normal mode)
1. Create RUN_DIR = `data/<today's date YYYY-MM-DD>_<OUR_TEAM>` (mkdir).
2. Delegate to the **data-scout** subagent for OUR_TEAM with RUN_DIR. It returns the
   EVENT_ID and the OPPONENT.
3. Then delegate to **two data-scout subagents at the same time (in one message, so they
   run in parallel)**:
   - one for OPPONENT with RUN_DIR and EVENT_ID,
   - one for OUR_TEAM with RUN_DIR and EVENT_ID, told to fetch the game summary and
     update facts_<OUR_TEAM>.md with leaders, injuries, odds and series info.
4. Confirm both facts sheets exist. Report any "Missing data" lines in one sentence.

## Step 1b — Research (SAVED MODE, for demos)
Skip downloading. Use Glob to find the most recent `data/*_<OUR_TEAM>` folder that has
two facts_*.md files, set RUN_DIR to it, and say which folder you are using.

## Step 2 — Draft
Delegate to **notes-writer** with RUN_DIR, both facts sheet paths and OUR_TEAM.

## Step 3 — Fact-check loop
Delegate to **fact-checker** with RUN_DIR.
- If STATUS is FAIL: send the ISSUES list to **notes-writer** to revise, then run
  **fact-checker** again. Maximum 2 revision rounds.
- Report each round in one line, e.g. "Fact-check round 1: 2 issues found, sent back to writer."
- If it still fails after 2 rounds, continue to Step 4 but show the unresolved issues clearly.

## Step 4 — Human approval (required)
Tell the editor: "Draft ready: open RUN_DIR/draft.md (Ctrl/Cmd+Shift+V for preview)".
Show the Quick Hits section and the fact-check result in the chat.
Then ask the editor to choose, using the question tool if available:
- **Approve and publish**
- **Request changes** (then ask what to change)
- **Cancel**

If changes are requested: send the feedback to **notes-writer**, then run Step 3 again,
then return to Step 4. Loop until the editor approves or cancels.
Never skip this step and never treat your own judgment as approval.

## Step 5 — Publish (only after explicit approval)
Let SLUG = `<week>_<AWAY>_at_<HOME>` (e.g. `wk4_NE_at_BUF`).
1. Copy the approved draft to `output/<SLUG>_notes.md`.
2. Create `output/<SLUG>_notes.html`: read `templates/notes-template.html`, replace
   `{{TITLE}}` with the notes title, `{{TEAM_COLOR}}` with OUR_TEAM's color hex from the
   facts sheet (with a leading #; use #1D5C3A if unknown), `{{GENERATED}}` with today's
   date, and `{{CONTENT}}` with the notes converted to clean HTML (h1/h2, ul/li, table,
   p, strong). Do not change the template's CSS.
3. Append a row to `output/approval_log.csv` (create it with a header row if missing):
   `timestamp,notes_file,team,approved_by,fact_check_status,fact_check_rounds,issues_fixed,run_dir`
   Use "Editor" for approved_by unless the editor gave a name.
4. Finish with a short summary: files created, fact-check rounds, issues fixed, and
   "Open output/<SLUG>_notes.html in your browser (right-click → Reveal in File Explorer/Finder)."

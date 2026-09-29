# Game Notes Agent: Build Guide (Claude Code in VS Code, no terminal, no Python)

You will build and run an agentic workflow for an NFL team's Communications department
entirely inside VS Code's Claude Code panel. The "code" is plain-English Markdown files
that define a workflow (a skill) and three specialist agents (subagents).

```
 /game-notes BUF
      │
      ▼
 Coordinator (the game-notes skill, in your chat)
      │
      ├─► data-scout (BUF) ─┐        downloads ESPN data, saves raw JSON,
      ├─► data-scout (OPP) ─┘ parallel  writes a facts sheet with sources
      │
      ├─► notes-writer            drafts notes from facts sheets ONLY (no web access)
      │
      ├─► fact-checker ──fail──►  back to notes-writer (max 2 rounds)
      │        │ pass
      ▼        ▼
 You, the editor: Approve / Request changes / Cancel
      │ approve
      ▼
 Publish: output/*.md + styled *.html + approval_log.csv
```

| File | What it is |
|---|---|
| `CLAUDE.md` | Project brief and accuracy rules. Claude Code reads it every session. |
| `.claude/skills/game-notes/SKILL.md` | The workflow. Typing `/game-notes BUF` runs it. |
| `.claude/agents/data-scout.md` | Research agent: downloads data, writes sourced facts sheets |
| `.claude/agents/notes-writer.md` | Writing agent: has no web access, can only use facts sheets |
| `.claude/agents/fact-checker.md` | Verification agent: read-only, checks every claim |
| `.claude/settings.json` | Pre-approved permissions (ESPN downloads, writing to data/ and output/) |
| `templates/notes-template.html` | The published notes design, colored with your team's color |

---

## Step 0: Prerequisites (20 min)

1. Install **VS Code**.
2. You need a Claude account that includes Claude Code (a paid Claude plan) or an Anthropic
   Console account. Check Anthropic's Claude Code docs for current plan details.
3. In VS Code, open the Extensions view (Ctrl/Cmd+Shift+X), search **Claude Code**, and
   install the extension published by Anthropic. Open it from the Claude icon in the sidebar
   and sign in.
4. Windows only: if Claude Code asks for Git for Windows, install it from git-scm.com and
   restart VS Code.

## Step 1: Open the project (5 min)

1. Unzip `game-notes-agent` and open the folder in VS Code (File → Open Folder).
2. When VS Code or Claude Code asks whether you trust the folder, choose **Trust**.
   (Project settings and agents only load in trusted folders.)
3. Open the Claude Code panel.

## Step 2: Have Claude Code explain the project (15 min)

Type this in the Claude Code panel:

> Explain how this project works, file by file, as if I'm presenting it to a class.
> Then list the subagents and the skill you can see.

You should see `data-scout`, `notes-writer`, `fact-checker` and the `/game-notes` skill.
If the agents aren't listed, close and reopen the Claude Code panel.

## Step 3: First real run (15 min)

Type:

> /game-notes BUF

(Use any team: KC, PHI, DET, SF, LAR, and so on.)

What you'll see:
- a short plan,
- data-scout agents downloading data (two run in parallel),
- the writer drafting, the fact-checker checking, and possibly a revision round,
- a question asking you to **Approve**, **Request changes**, or **Cancel**.

Before answering, open `data/<date>_BUF/draft.md` and press Ctrl/Cmd+Shift+V for a
formatted preview. Try **Request changes** once (e.g. "Lead with the run game and make
the social posts shorter") to see the loop, then **Approve**.

Permission prompts: Claude Code may ask before running commands or writing files. Choose
the option that allows it for this project. `.claude/settings.json` pre-approves the
common ones.

When it finishes, find `output/` in the Explorer, right-click the `.html` file →
**Reveal in File Explorer/Finder**, and double-click to open it in your browser.

## Step 4: Make it yours (1-3 hours)

Your instructor will want to see your own work. Ask Claude Code to make changes for you;
you never edit code by hand. Good upgrades:

- **Game-day conditions:**
  > Update the data-scout agent to also download
  > https://site.api.espn.com/apis/site/v2/sports/football/nfl/scoreboard, find our game,
  > and add weather, broadcast network and venue details to the facts sheet.
  > Add a "Game Day" section to the notes-writer's format.
- **A fourth agent:**
  > Create a social-media subagent in .claude/agents that writes 3 posts for X, 2 for
  > Instagram and 1 for TikTok from the approved notes, and add it as a step after
  > approval in the game-notes skill.
- **A second audience:**
  > Add an option to /game-notes so "/game-notes BUF coaches" produces a short opponent
  > scouting brief for the coaching staff instead of media notes.

After each change, run `/game-notes` again to test it.

### If your instructor expects you to build every file with Claude Code
Start from an empty folder that contains only `CLAUDE.md`, then give Claude Code these
prompts in order, reviewing each file it creates (the files in this kit are the answer key):
1. "Read CLAUDE.md. Create the data-scout subagent it describes in .claude/agents, using
   ESPN's public site API endpoints for team info, schedule, game summary and season stats.
   It must save raw JSON with curl and write a facts sheet where every line cites its file."
2. "Create the notes-writer subagent. It may only use Read, Write, Edit and Glob, so it has
   no web access, and it writes draft.md in a fixed game-notes format."
3. "Create a read-only fact-checker subagent that checks every claim in draft.md against
   the facts sheets and raw JSON and replies with STATUS: PASS or FAIL and a numbered issue list."
4. "Create a /game-notes skill that runs the scouts in parallel, then the writer, then a
   fact-check loop of at most 2 rounds, then asks me to approve, request changes or cancel,
   and only after approval publishes Markdown, an HTML version and an approval log row."
5. "Create .claude/settings.json allowing curl and ESPN WebFetch, and edits in data/ and output/."

## Step 5: Measure it (1 hour)

Run the workflow for 5-10 different teams and approve each. `output/approval_log.csv`
records the fact-check status, rounds and issues fixed for every run. Also record:
- **Time:** stopwatch each run, versus a manual baseline (time yourself building one packet
  by hand from ESPN and Pro Football Reference).
- **Accuracy:** issues the fact-checker caught; spot-check 3 numbers yourself on ESPN.com.
- **Cost:** check your Claude usage page, or type `/cost` in the Claude Code panel if you use an API account.

## Step 6: Demo-proof it (night before)

1. Run `/game-notes BUF` (your demo team) the day before. The raw data and facts sheets
   stay in `data/`.
2. For the live demo, run **`/game-notes BUF saved`**. It skips downloading and uses the
   saved data, so ESPN outages or a changed schedule can't break your demo. Mention in the
   presentation that live mode exists and show the raw files it downloaded.
3. Rehearse three times. A run takes a few minutes; practice narrating each agent.
4. Screen-record one full successful run as a backup.
5. Keep a finished `output/*.html` open in a browser tab as a last resort.
6. Presentation day: laptop plugged in, notifications off, VS Code zoomed in (Ctrl/Cmd +).

## Step 7: Publish to GitHub (10 min, no terminal)

1. Create a free account at github.com. Install Git from git-scm.com if VS Code asks.
2. Open **Source Control** (Ctrl/Cmd+Shift+G) → **Publish to GitHub** → sign in →
   **Publish to GitHub public repository**.
3. There is no API key in this project (Claude Code handles sign-in), and `.gitignore`
   excludes run data and outputs. To include one sample output in the repo, ask Claude Code:
   "Copy the latest output HTML into a new examples/ folder."

## Step 8: The 12-minute presentation

| Time | Show | Say |
|---|---|---|
| 0:00-1:30 | Problem | Team PR staff rebuild game notes by hand every week from many sources. |
| 1:30-3:00 | Diagram (top of this guide) | Four roles, each an agent with limited tools; a human approves. |
| 3:00-4:00 | `.claude/agents/notes-writer.md` | "The writer can't browse the web. It can only use verified facts." |
| 4:00-9:00 | **Live demo** `/game-notes BUF saved` | Narrate each agent. Show the fact-check result. Request one change. Approve. Open the HTML. |
| 9:00-10:30 | Results | Time, accuracy and fact-check numbers from your approval log. |
| 10:30-12:00 | Limits + next steps | Unofficial data source, human stays accountable, future agents. |

**Likely Q&A**
- *How do you prevent made-up stats?* Every number is copied from saved raw data into a
  sourced facts sheet; the writer has no web access; a separate read-only agent checks
  every claim and spot-checks the raw files; a human approves.
- *Why is this agentic and not a script?* The coordinator delegates to specialists that
  choose their own steps, work in parallel, and revise based on feedback.
- *Where's the code?* The workflow is defined in natural language plus tool permissions.
  That's the design choice: a PR department could maintain it without engineers.
- *What if ESPN changes?* The endpoints are public but unofficial. The scout records
  missing data instead of guessing, and saved mode keeps the demo reliable.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `/game-notes` not found | Make sure the folder is trusted and `.claude/skills/game-notes/SKILL.md` exists; reopen the Claude panel. |
| Agents not listed | Reopen the Claude panel. Check each agent file starts with `---` on line 1. |
| curl fails on Windows | The scout retries with `curl.exe`. If downloads still fail, tell Claude: "use WebFetch instead of curl". |
| Endless permission prompts | Choose "allow for this project", or switch the panel to accept-edits mode. |
| Fact-check never passes | Read the issues. Usually a missing field; ask Claude Code to improve the data-scout instructions. |
| Wrong or no upcoming game | Bye week, or the season's last game passed. Try another team. |

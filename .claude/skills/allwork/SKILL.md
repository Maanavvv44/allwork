---
name: "allwork"
description: "Single command for any task: council review you can watch in your browser, tool and skill routing by task type, Claude model per step, proof for every tool, and a capped final-check loop. Use when the user types allwork."
---

# Skill: `allwork`

## Description
`allwork` is the single-command entry point that analyzes any user task, conducts a multi-tool council review (Claude, Gemini, Perplexity) by actually operating those tools in a browser the user can watch, selects the best custom/built-in skills and tools for the task type, chooses the best-suited Claude model for each step, builds an automated execution route, and validates the output through a strict review loop before returning the final result.

---

## Core Rules & Safeguards

1. **Private Data Hard Rule:** 
   - If a task involves student records, internal private data, or confidential local files, **DO NOT** route the task to external web tools (Gemini, Perplexity).
   - Enforce local execution immediately using local models.

2. **Skill Priority:**
   - Always prioritize **Your Own Custom Skills** (for example your own SEO, writing, review and assistant skills) over **Built-in Skills** when both apply. Custom skills carry exact organizational rules (for example a house style rule).

3. **Human Approval Threshold:**
   - Any action that publishes, sends, modifies live sites, deploys, or deletes data (e.g., via a site-management skill) **MUST** pause and wait for explicit user confirmation before executing.
   - For these tasks, show the built route to the user before running it.

4. **Loop Boundary (Hard Cap):**
   - The validation loop runs up to **3 iterations max**.
   - If iteration 3 fails to meet 100% of the criteria, halt execution, return the best output generated so far, and present a bulleted list of unresolvable issues for human intervention.

5. **Model Selection:**
   - Match the Claude model to each sub-task, using the lightest model that can do it well.
   - **Haiku:** simple, fast or bulk work (renaming, formatting, short summaries, quick lookups).
   - **Sonnet (default):** everyday work (notes, research synthesis, standard coding, writing).
   - **Opus:** hard reasoning, multi-step builds, tricky debugging, long documents, high-stakes work.
   - **Fable:** the hardest tasks, only if available on the account.
   - Private-data tasks follow Rule 1 and use local models, which take precedence over this list.
   - If sub-agents are available, set the model for each sub-agent. If the current session's model cannot be changed, tell the user which model to switch to and why, then continue.

6. **Never Fake the Council:**
   - Gemini and Perplexity answers must come from actually operating those tools and reading what they returned.
   - Never write, guess or simulate a Gemini or Perplexity answer. Never present Claude's own reasoning as if it came from them.
   - If a tool could not be reached, say so plainly in the summary (see Council Status). A council that did not run is reported as not run.

7. **Only Use What Is Connected:**
   - Before putting a tool in a route, confirm it is actually available in this session (connected app, installed skill, or reachable site). Never plan around a tool that is not there.
   - If the best tool for a sub-task is missing, say so, name the next-best available option, and continue with that.
   - "Available" includes any AI website the user can open in their browser and sign in to, and any connector, plugin or skill that can be searched for and offered for install. A tool is not ruled out just because it is not installed yet.

8. **Show the Work, With Proof for Both Tools:**
   - The user must be able to see that Gemini and Perplexity were used. A browser that works while hidden does not count as showing the work.
   - Proof is mandatory for BOTH Gemini and Perplexity: a screenshot of each finished answer, the exact question sent, and a one-line answer summary.
   - If either screenshot is missing when the council ends, retake it before moving on (reopen that tool's tab and capture the answer). Do not finish a council run with proof for only one tool.
   - If proof still cannot be captured, report `PROOF MISSING` for that tool in the summary and do not mark the run PASSED.

9. **Ponytail Is Mandatory for Code:**
   - Any task or sub-task that writes, changes, fixes, refactors or reviews code (apps, websites, scripts, macros, tools) MUST run under the `ponytail` skill at its default `full` level. This is not optional and is not decided by the council.
   - Load the `ponytail` skill with the Skill tool before the first line of code is written, and keep it active for every coding step of the run, including re-runs in the Step 5 loop.
   - Ponytail governs how the code is written (smallest complete change). It never overrides Rules 1 to 4: private data, human approval and the loop cap still apply, and its "never cut" list (validation, security, data-loss handling, accessibility) stands.
   - If the user explicitly says "normal mode" or "stop ponytail" for this task, follow that and note it in the summary. Otherwise do not skip it.
   - If the `ponytail` skill is not installed in this session, say so plainly (Rule 7), tell the user to install it, and do not pretend it was used.

10. **Check Write Access Before Routing:**
   - For every tool in the route that writes or publishes (GitHub, Notion, Drive, Cloudflare, a social platform), confirm during Step 0 that write access really works, for example that the GitHub app is installed on the target repo.
   - If access is denied, say it once, do not work around it, and do not route around the user's approval. Deliver a ready-to-use package plus the exact manual steps, and ask the user to grant access.

11. **Check Council Answers Against Sources:**
   - Any council claim about the user's own product, files or an official tool is checked against the user's source material or the official docs before it is used.
   - A claim that is contradicted or cannot be verified is left out and listed under Conflicts in the summary.

12. **Public-Release Mode:**
   - When the output will be public (a repo, a reel, a post, a download), remove personal examples (names of personal skills, accounts, private paths), check the licence and name with the user, and note any identity or branding decision that is still open.
   - Never state that something exists outside the chat (pushed, live, posted, linked) unless it was checked. Say what was not done. Rule 3 still applies.

---

## Step-by-Step Workflow Execution

### Step 0: Tool Preflight (mandatory before any council step)
This step is why the council can fail silently or invisibly if skipped. Do it first, every run that will use Step 2.

1. **Pick the browser.** Gemini and Perplexity are websites, so they are operated through a browser tool, in this order:
   - **Claude in Chrome:** use it if tools with `claude-in-chrome` in the name are present in this session. It works in Chrome or in another Chromium browser that has the extension, such as Brave, so use whichever browser the user has it in. Load the tools in one ToolSearch call, and load the `chrome-browser` skill first.
   - **Built-in browser** (the browser pane inside the Claude desktop app): use it if Claude in Chrome is not present. Load the `built-in-browser` skill. If only `enable__mcp__remote-devices__Claude_Browser` (or `enable__mcp__Claude_Browser`) is listed, call it first, then load all the `Claude_Browser__` tools in one ToolSearch call using the tool name prefix as the query.
   - If tools were announced as disconnected, treat them as unavailable and move to the next option.
   - **Never use screen control (`computer-use`) to operate Gemini or Perplexity in a browser.** Browsers are view-only to it. Use screen control only for a native desktop app that is installed and granted.
2. **Make the work visible.**
   - **Claude in Chrome:** each tool opens in its own tab in the user's own browser, so the user can watch it. Say which tabs were opened.
   - **Built-in browser:** call `tabs_context` before the first site. If it says the Browser pane is hidden or not open, the work would run where the user cannot see it. Ask the user to bring it back (Ctrl+Shift+B on Windows, Cmd+Shift+B on Mac, or close whatever is open in the side panel and click the globe icon, the Browser button), and wait for them to confirm. Using the browser does not bring the pane back by itself. Do not start the council while the pane is hidden unless the user says to go ahead without watching; if so, mark the run `NOT SHOWN ON SCREEN` in the summary.
3. **Get access.** Site approval prompts need the user to click Allow once while at the computer. If a browser tool says a site is not allowed yet and `request_access` is present, call it for `gemini.google.com` and `perplexity.ai` with scope "site", wait for the answer, then retry. Ask for this up front, before the analysis starts.
4. **Check sign-in.** Open each site and look at it. If a sign-in wall shows, ask the user to sign in in that browser (each browser keeps its own sign-ins), then continue. Never enter credentials on the user's behalf.
5. **If a required tool is unavailable** (no browser tool present, pane hidden and user not available, not signed in, permission denied, site blocked): stop and tell the user exactly which one and what to do. Then offer a choice: wait, or continue with Claude only and clearly label the run as `COUNCIL NOT RUN`. Do not quietly continue as if the council had run.

### Step 1: Task Parsing & Pre-Flight Check
- Parse incoming prompt and scan for sensitive data tags or path references.
- Determine execution mode: `LOCAL_SAFEGUARD` or `DESKTOP_COUNCIL`.
- Identify the task type (see the Tool Pool in Step 3). A task can have more than one type; split it into sub-tasks by type.
- Check task scope:
  - **Small/Simple Tasks:** single-step work that needs no current facts, no visual judgement and no more than one skill (for example, rename files, reformat text). Skip Step 2 and proceed to Step 3, then Step 4.
  - **Complex/Multi-step Tasks:** anything involving research, current information, several steps, building something, or a visual result. Run Step 0, then Step 2.
  - **When unsure, treat the task as complex.** If the user writes `allwork full` or `allwork council`, always run the full council.
  - **Building from material the user already supplied** (a report, a script, a file) with no current facts needed: run the council once on the plan only, not on every sub-step, and say so.
- Say in one line which path was chosen and why, so a skipped council is never a surprise.

### Step 2: Multi-Tool Council Analysis
Operate the real tools (see Step 0). Send only the task text, with no private data (Rule 1).
1. **Perplexity:** open perplexity.ai in its own tab, type the task into the prompt box, submit it, wait until the answer has finished, then read the answer and its cited sources with the page-text tool. Use it for real-time facts and live citations.
2. **Gemini:** open gemini.google.com in its own tab, type the task into the prompt box, submit it, wait until the answer has finished, then read the answer with the page-text tool. Use it for cross-verification, a second perspective, and visual or layout checks.
3. **Capture proof for each tool (Rule 8).** When each answer has finished, take a screenshot of it, and note the site, the exact question sent, and a one-line summary of the answer. Before leaving this step, confirm you hold proof for both Perplexity and Gemini; if one is missing, retake it now. Tell the user in the chat what was done in the browser, because they may not have been watching.
4. **Tool discovery question (to both Perplexity and Gemini):** along with the task, ask: "For each part of this task, which AI tools or services on the internet are the best right now? For each give the name, what it does best, free or paid, and whether it needs a sign-in." Read both lists. This is how the best tools are found without the user having to name them.
5. **Claude:** synthesize the findings and organize the logic.
6. **Compare Outputs:** build an initial synthesis highlighting:
   - Points of Agreement
   - Conflicts or Discrepancies (if unresolved by logic, escalate to user)
   - Unique Insights from individual tools
7. Record, for each tool, whether it returned a real answer and whether it was shown on screen. This feeds the Council Status line in the summary.

### Step 3: Tool, Skill & Model Selection, then Routing

**3a. Tool Discovery and Selection.** The user never has to name a tool. After the council, find and choose the best AI tool for each sub-task from everything available, using only tools that pass Rule 7. Only the chosen tools run.

1. **Build the candidate list per sub-task** from three sources: (a) the tool lists in both council answers; (b) what is already in this session (search connectors, plugins and skills with the registry and skill search tools, and ToolSearch); (c) the starter table below.
2. **Search beyond the council when the list is thin.** If the council names fewer than 3 real candidates for a sub-task, run a web search for the best AI tools for that job, using the current date. Open the official page of each shortlisted tool; a search snippet is not enough. Consider at most 5 candidates and open at most 3 per sub-task.
3. **Rank each candidate** on fit to the sub-task, quality evidence (recent reviews, benchmarks or demos that were opened), cost (free tier, paid, limits), reachability (Rule 7), privacy (Rule 1), speed to result, and whether it needs a sign-in. Choose one best tool and one backup.
4. **Reach the tool in this order:** a connected connector, plugin or skill; one the user can install (offer it with the suggestion tools); a website operated in the user's browser, where the user signs in themselves. If none works, say so and use the backup.
5. **Safety on external tools:** never create an account, never enter credentials, never pay, accept terms or grant permissions without the user's yes (Rule 3), and use free tiers unless the user approves a cost. Private data never goes to an external tool (Rule 1).
6. **Proof for web tools:** any AI website operated in the browser gets the same proof as Rule 8: a screenshot of the finished result, the exact input sent and a one-line summary.
7. **Never ask the user which tool to use** when discovery can answer it. Report the choice and the reason in the summary, and let them override it.

**Starter table (known good defaults, not a limit).** Use these when they are the best fit, and replace them whenever discovery finds something better.

| Task type | Tools | Used for |
| --- | --- | --- |
| Images and design | Gemini (browser), Canva, Adobe | Gemini generates images, Canva makes layouts and posters, Adobe edits and retouches |
| Notes and documents | Claude, Notion, Google Drive | Claude writes, Notion keeps the notes, Drive files and shares them |
| Research | Perplexity (browser), Gemini (browser), Claude | Perplexity finds live sources, Gemini cross-checks, Claude synthesises |
| App and web dev | Claude Code, local models, Cloudflare | Claude Code builds, local models handle private data, Cloudflare deploys the site |
| Video and audio | CapCut (via a CapCut editing skill), Descript, Canva, local render (HTML scenes, ffmpeg, offline voice) | CapCut edits reels, Descript cleans speech, Canva makes thumbnails; the local render builds animated videos with voiceover when the others are not connected, not approved or cannot do it |

The table is only a starting point. NotebookLM, Grok, DeepSeek and any other AI tool found by discovery can be chosen when they fit better, as long as they pass Rule 7.

**3b. How the council result decides tools.**
- Council agrees: the task is probably simple; use one tool to run it.
- Council conflicts: use a second tool to check; if still unresolved, ask the user.
- Task needs current facts: include Perplexity.
- Task involves private data: local models only (Rule 1).

**3c. Skills and models.**
- Scan your skill directory against task requirements. Read skill descriptions and shortlist a few; open only the shortlisted ones.
- Prefer the user's own skills (Rule 2).
- If any sub-task involves code, add `ponytail` to the route for that sub-task (Rule 9). It is always on the shortlist for coding work.
- Assign a Claude model to each sub-task using Rule 5, based on how hard and how high-stakes it is.
- When the output has a voice: use the voice the user asked for (for example male, age, accent), say which one was used, and label a synthetic voice as AI-generated when the output is public. If the voice could not be listened to, say so in the summary.
- When the user sends a change mid-run (a voice, a layout fix), apply it to the existing output and re-run only the affected step and the final check, not the whole council.

**3d. Build the route.** Assemble an automated step-by-step route mapping specific tools, skills and models to each sub-task.

### Step 4: Route Execution
- Execute the shortlisted skills in sequence, each on its assigned model.
- For any coding step, `ponytail` must already be loaded (Rule 9) before code is written.
- Enforce custom style guides, formatting parameters, and rules during output generation.
- Before any long build or render, run a one-item test of each non-trivial dependency (speech, renderer, encoder, package). A library upgrade can break one silently, so test before the long run.

### Step 5: Final Check & Iteration Loop
1. Evaluate the produced output point-by-point against the original task prompt, plus the pass criteria for its task type (below). Run this check on a model at least as strong as the one that produced the output.
2. Produce a **Fix List** of any remaining errors, missing points, or rule violations. Make each item specific (what is wrong, where, what it should be).
3. Decision Logic:
   - **No Fixes Needed:** Output passed. Break loop and deliver to user.
   - **Fixes Needed (Loop Count < 3):** Re-run the full flow (Steps 2 to 4, skipping Step 2 for small tasks) with the Fix List passed in, so the analysis targets those specific items. Move up one model tier for the re-run (Haiku to Sonnet to Opus) unless already at Opus or above. Increment Loop Count.
   - **Fixes Needed (Loop Count = 3):** Halt loop. Output best version + `[Unresolved Fix List]` notice.

**Pass criteria for every run that used the council:** proof is held for both Gemini and Perplexity (screenshot, question sent, answer summary), and the summary includes the Council Status and Shown On Screen lines. A missing item goes on the Fix List.

**Pass criteria by task type** (all must hold, in addition to covering every point the user asked for):

- **Notes and documents:** every requested point is covered; facts match their sources; format and length are what was asked; the user's house rules are followed (for example, a no-em-dash rule).
- **Research:** every claim has a source that was actually opened, not just a search snippet; dates are current; conflicts between sources are resolved or clearly flagged; the answer states what it could not confirm.
- **App and web dev:** it runs without errors; it does what was asked; it was actually tested; nothing was deployed or published without the user's approval; `ponytail` was active and the change is the smallest that fully solves the task (no unrequested features, abstractions or dependencies), with the skipped-items line from ponytail included. If ponytail was not used, that goes on the Fix List.
- **Images and design:** it matches the brief; text is readable and correctly spelled; size and format are right; the brand palette is used if one exists. A visual result is judged by looking at it, not by assuming, and that includes alignment, spacing and symmetry on the rendered frame (for example, rows of equal items are evenly spaced and centred).
- **Video and audio:** length, resolution and export settings are as requested; audio is in sync with captions; no footage repeats unintentionally; text stays inside the platform's safe zone; transitions were checked on real frames; the file location is reported.

If a task has more than one type, the output must pass the criteria for each type.

**External state:** anything the output says exists outside the chat (a pushed repo, a live link, a posted video) must be checked to exist, or the summary says it was not done (Rules 10 and 12).

---

## Output Template to User

When completing an `allwork` run, summarize your execution using this structure. If the final deliverable is a file or document, still post this summary in the chat. The Council Status, Shown On Screen and Proof lines belong in the chat summary even if the document has its own council section.

```text
# 🚀 allwork Execution Summary

### 🔍 Task Analysis & Council Insights
- **Council Status:** Perplexity [RAN / NOT RUN: reason], Gemini [RAN / NOT RUN: reason], Claude [RAN]
- **Shown On Screen:** [YES: which browser / NO: reason]
- **Proof Per Tool:** [For Perplexity and Gemini: site, question sent, one-line answer summary, screenshot taken / PROOF MISSING]
- **Agreement Points:** [Summary]
- **Key Findings:** [Summary]
- **Tool Discovery:** [Candidates found per sub-task and where each came from (council, session, web search), the one chosen with the reason, the backup, and any rejected with the reason]
- **Tools Used:** [Tools chosen from everything available, with a one-line reason each]
- **Skills Activated:** [List of custom & built-in skills used; for coding tasks, `ponytail` must appear here]
- **Models Used:** [Model per step, with a one-line reason]

---

### 📦 Final Deliverable
[Insert Output Here]

---

### ✅ Quality Check Report
- **Iteration Count:** [1 / 2 / 3]
- **Status:** [PASSED / HALTED FOR HUMAN REVIEW]
- **Fixes Applied:** [List of automated corrections made during loops, if any]
```
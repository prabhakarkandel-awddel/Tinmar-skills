---
name: dev-tickets-pr-workflow
description: Full dev loop the user runs — problem intake (plan directory + feat/fix branch) → git pull (user) → conflict resolution → analysis → create Linear tickets → move them on cue → solve → strike in Linear → progress doc → Gmail draft to prasanna → status sync (in-progress/assigned to me) → commit → push → PR (+ comment). Use when user says "create tickets", "move the tickets", "solve them", "strike in linear", "create a plan/plan directory", "here's the problem", "do the workflow", "commit push PR", or hands over a pulled branch with conflicts.
---

# Dev → tickets → doc → email → PR workflow

This is ONE continuous loop the user drives in stages. Know the whole sequence
so you can act on any stage and anticipate the next. Tool mechanics (Composio,
Gmail, GitHub, Linear, docx) live in the companion skill
`composio-automation` — load/use it for the tool-level detail; this skill owns
the ORDER and the hand-offs.

## The sequence (canonical)

```
 0. problem stated    → plan directory (3 .md, gitignored) + branch feat/... or fix/...
 1. git pull          → USER does this (you never run it for them)
 2. conflicts         → USER hands them to YOU; resolve, verify, DON'T commit
 3. analysis          → scope the work from the conflict/branch context
 4. create tickets    → Linear; return the issue LINKS
    (alt intake 4b: user asks YOU to search Linear — "find urgent tickets
     related to X/Y" (+ optional status) → assign to user, cycle-move if user
     named cycle → full solve→strike→doc→email→PR loop on them)
 5. "move them"         → on user's cue: Todo → In Progress (their cue, not yours)
 6. solve tickets       → code work, per ticket, tests green as you go
 7. strike in Linear    → solved tickets: status → Done + short completion comment
 8. create a doc        → progress/deliverable .docx (see composio-automation §4)
 9. draft email         → to prasanna, doc attached, DRAFT ONLY
10. status sync         → tickets still open: In Progress, assigned to user
11. commit              → technical message (see below)
12. push                → origin, same branch
13. PR                  → populated description + "what was done" comment
```

User narrates the stages out of order ("create tickets", "tell you to move
them", "strike them", "now commit") — map each utterance to its stage and do
only that stage until told to advance. Stages 8–9 may also be requested
standalone.

## Porting work that already exists elsewhere (re-cut, dropped PR, external fix)

Used when asked to land work that lives in a held branch, a friend's diff, an
old fix, or an unmerged PR. Learn this once — do not re-derive it from the repo
each session.

1. **The spec lives on the ticket, not in the code.** Read the ticket's own
   paste-ready comment BEFORE writing anything; it supersedes the ticket
   description (descriptions go stale). It normally names the exact PR title,
   a **Keep** list, a **Drop** list, and which acceptance criteria are expected
   to come out NOT MET. The Keep/Drop lists ARE the scope — follow them
   literally, including the items that turn out to be already satisfied.
2. **Establish diff polarity before touching anything.** A supplied diff is
   `git diff A B`; files only in A show as `deleted file mode`. Read one hunk to
   learn which side is the fix, confirm with `git hash-object <file>` against the
   `index a..b` lines, and locate the source commit with
   `git log --all --find-object=<a-sha>`. Applying a fix→main diff **forward**
   removes the feature. Third-party patches may also be UTF-16LE — read them
   with `[IO.File]::ReadAllText($p, [Text.Encoding]::Unicode)`; `git apply`
   cannot take them.
3. **Base off fresh, one branch per unit.** `git switch -c <branch> <base>`
   (base = whatever the user names, usually `origin/main`), then
   `git rev-parse HEAD` must equal the base SHA — zero commits on the new
   branch. Never carry another unit's commits onto it, and never stack units.
4. **Atomic cut: one PR = one unit.** Port hunks BY HAND, only the files the
   Keep list names. Never merge / cherry-pick / revert a whole commit — it drags
   the companion work bundled into it. Before a verbatim file restore, prove no
   drift: `git log <base>..HEAD -- <path>` must be empty. Before committing,
   `git diff --cached --stat` must show exactly the expected file count.
5. **Everything outside the Keep list is excluded, and the PR says so.** Name
   the excluded files and the reason. Work that merely shares a commit with the
   fix (fonts, bundled assets, other generators) belongs to its own ticket.
6. **Title = the ticket's exact wording.** Specs frequently dictate it verbatim.
7. **Never fake green.** Acceptance criteria the chosen scope does not meet get
   marked **NOT MET** in the PR body with the reason and the proposed follow-up.
   A silent omission is a rejected review.
8. **Gates before push:** `ruff check` on every changed `.py`; targeted unit
   tests; `check:dashes` + `tsc --noEmit` if frontend touched; and a stash A/B
   (`git stash push -m <tag> -- <source files>` → rerun → `git stash pop`) for
   every failure before calling it a regression. Report failures as *triaged*,
   never hidden.

## Stage details

### 0. Problem intake → plan directory + branch (first thing for any new effort)
- **Trigger:** user describes a problem/quality effort ("OCR is bad", "agent
  reliability", "<topic> work") → set up the working area BEFORE any code.
- **Plan directory** at repo root, named from the topic in CAPS with a
  `-QUALITY/` (or similar) suffix — pattern precedent: `AGENT-QUALITY/`,
  `OCR-QUALITY/`. Exactly **three `.md` files** inside:
  1. **`PLAN.md`** — the tickets (IDs + links once created, strike-through
     when done) + high-level systems thinking: approach, phases, sequencing,
     acceptance criteria. The strategy document — gives DIRECTION.
  2. **`TECHNICAL.md`** — architecture + the code as it currently is: module
     maps, data flow, key files/functions/config keys, constraints, plus a
     **repo map** — where each functionality/scope of the work lives
     (`<area>` → `path/file.py:fn`, routes, tables, tests). The
     "how it works today + where to edit" reference — gives IMPLEMENTATION
     guidance.
  3. **`CONTEXT.md`** — discoveries + progress: append-only dated session
     entries (what was tried, what broke, decisions, pitfalls found), plus
     accumulated discoveries. Never rewrite history here, only append.
     **Major-sweep strike:** after every major sweep (a phase / batch of
     tickets lands), strike the completed session entries and roll them up —
     keep a compact standing block of **discoveries, gotchas, current
     progress status, and the updated repo map**, discarding the day-to-day
     narration that's now encoded in code/PLAN. The file stays lean and
     session-zero-readable instead of growing into an unread log.
- **Purpose (why these exist): token conservation without quality loss.**
  Every future session reads the three files FIRST instead of re-exploring the
  repo — the directory is a cached map of direction, implementation and
  location. Maintain it as you work: new file/function touched → add its path
  to the repo map; decision made → CONTEXT entry; scope shifted → PLAN update.
  Stale docs defeat the purpose — update in the same change as the work.
  A session that must grep the whole tree to rediscover what PLAN/TECHNICAL
  already answer is wasted tokens.
- **Gitignore it:** add `<DIRNAME>/` to `.gitignore` in the same change
  (precedent: `AGENT-QUALITY/`, `OCR-QUALITY/` entries) — planning docs are
  cross-session local state, never committed.
- **Branch it:** create + checkout a properly named branch for the effort —
  `feat/<TOPIC>` or `fix/<TOPIC>` depending on whether it's new work or a
  repair (e.g. problem `OCR-QUALITY` → branch `feat/OCR-QUALITY`,
  `fix/agent-quality` precedent). Tell the user the branch name. All later
  stages (solve → commit → push → PR) ride THIS branch; the PR base is whatever
  branch the user names (never assume `main`).
- Cross-links: ticket links from stage 4 land in `PLAN.md`; strikes are
  mirrored there; discoveries flow into `CONTEXT.md` as you work.

### 2. Conflict resolution (ask-first)
- Never commit or push after resolving — user reviews first.
- Understand BOTH sides before editing (often: our OCR/agent change vs an
  incoming integration merge). Keep both intents unless they're semantically
  exclusive — then ASK which wins.
- After resolving: run the affected tests + `ruff check` on touched files;
  report what conflicted and which strategy you chose, file by file.

### 4. Create tickets
- Structure: one parent epic + child tickets per unit of work.
- **Naming: descriptive, never T-numbers or local IDs.** Ticket titles say
  what the work IS ("Pooled Redis clients in hot sync paths"), not "T1"/"T2".
  Group with **labels** instead (e.g. `speed`, `rework`) — create the label
  on the team if it doesn't exist. The same rule extends to **commits, PR
  titles and PR bodies: no T-numbers / local ticket ids anywhere** —
  describe the change in plain words; a Linear link (LIN-x) in a PR body
  is fine, a bare local "T3" is not. Keep local PLAN.md bullets keyed by
  the same descriptive names so nothing reintroduces the numbers.
- Each ticket: actionable title, enough body that a future agent can execute
  without re-deriving context (file paths, pitfall IDs, acceptance line).
- **Return the Linear issue links** in chat, one per line.
- Linear seat/credit limit may block creation (`USAGE_LIMIT_EXCEEDED`): report,
  cache every resolved ID (team/cycle/assignee/status-column UUIDs — status
  columns for Todo and In Progress; assignee = the user) so the retry is a
  one-shot, and let the user lift the limit.
- Cross-reference tickets in the local planning docs (same bullets, struck
  when done) — local docs are your source of truth while Linear is blocked.

### 4b. Ticket search intake (user asks YOU to find them)
- User does NOT search manually — they ask opencode (via Composio/Linear) to
  find work: *"find tickets related to X, Y that are urgent"*, optionally with
  a status filter (*"…that are still Todo"*, *"unassigned"*).
- Search: query by keywords/labels matching the ask + every qualifier the user
  said (urgency/priority, status, assignee-unassigned, team). Return the match
  list as **Linear links with titles**, note priority/status per row, and
  confirm scope before mass-changing anything (>5 tickets or ambiguous match →
  ask which to take).
- For each selected ticket (usually the unassigned/picked ones):
  1. **Assign to the user** (assignee = me/user).
  2. **Move to cycle ONLY if the user names the cycle in the prompt**
     ("move them to cycle 7", "cycle x") — typically tickets sitting in
     Backlog; cycle id is never guessed, no cycle mentioned → leave cycle as-is.
     Already-in-active-cycle tickets: don't re-cycle unless told.
  3. Status stays as-is until the user's usual cue ("move them" → In Progress).
- Then run the FULL loop on them like any other ticket: solve → strike (Done +
  completion comment) → included in the progress doc → email draft → status
  sync → technical commit → push → PR (description + "what was done" comment).
- Confirm after intake: links + assignee/cycle changes applied, then start work.

### 5. "Move them" / "strike in Linear"
- These are USER-triggered transitions — wait for the explicit cue.
  - *move them* = Todo → In Progress (newly started work).
  - *strike in Linear* = ticket solved → Done + one-line completion comment
    (what changed, where).
  - *status sync (stage 10)* = any ticket not actually finished stays
    **In Progress, assigned to the user** ("assigned to me") — never mark
    unfinished work Done to look tidy.
- Mirror every transition into the local planning docs (strike the line) in
  the same breath.
- Only this agent may transition Linear issues (MCP), but editing local docs
  is always free.

### 8–9. Doc + email
- **ALWAYS create a .docx progress report and attach it to the Gmail draft.**
  The doc lists action items only — what was done, in plain language.
  **Never mention PRs, issue/ticket creation, test runs, or CI in the doc or
  the email body.** Just the code changes, cleanly formatted.
- **Doc format:** heading "Progress Report" (never a date in the heading —
  the date lives only in the file name, `Progress Report - <date>.docx`),
  then "Task completed today: <one-line summary>", then one section per unit
  of work:
  bold title (what was done, not ticket IDs), one plain-language paragraph
  explaining the change and its effect. Close with a one-line summary.
  Generate with python-docx inside the Composio workbench sandbox, upload
  via `upload_local_file`, use the returned `s3key` as the Gmail attachment.
- **Email body stays SHORT — the task list goes in the attachment, never in
  the mail body.** **EVERY mail to prasanna going forward uses this exact
  format (user directive, 2026-10-26 — supersedes any earlier body):**
  - Subject: `Progress report for today` — **never a date in the subject or
    the body; the date lives only in the attachment's file name**
    (`Progress Report - <date>.docx`).
  - Body, verbatim and nothing else: `Respected, sir please review my work
    for the day`
  - The docx attachment carries everything else. No numbered bullets, no
    summary paragraphs, no PR/issue/test mentions in the email itself.
- Recipient is a NAME ("prasanna"), never an address → loose People search,
  domain tag `@awddel.com`, tie-break by people already mailed
  (`composio-automation` §1 lookup rules). Subject = the user's wording.
- **Draft, not send** — unless the user explicitly says "send it"; then send
  AS-IS (verify recipient only, never re-edit their hand adjustments).
  Show the draft/sent link. Delete the local doc afterwards when asked.

### 11. Commit — technical message, no exceptions
- Subject: `area: imperative summary` ≤72 chars.
- Body: per-change paragraphs with real substance — paths, functions, config
  keys, metric names, ticket/pitfall IDs, WHY, deliberate exclusions + reason,
  test counts and failure triage. Aim: a future agent fully understands the
  change from `git log` alone.
- PowerShell BOM trap + message-file handling: `composio-automation` §2.
- Stage deliberately: `git add -u` + explicit new files; before adding any
  untracked file, check WHY it's untracked (held-back goldens, stray docs).
  Never sweep unrelated files into the commit.

### 12. Push
- `git push origin <same-branch>` — never rename the branch, never force-push.

### 13. PR (+ comment)
- Base branch **as the user names it** (they will often say
  `integration/distributor-network`, NOT `main`) — confirm it back in your
  summary. Never default to `main`.
- Description populated from the real diff + commits (Summary / What changed /
  Test plan / Notes-exclusions template in `composio-automation` §7).
- **PR description content (user directive, 2026-10-15):** a proper description
  spells out three things **in this order, as flowing text — never as
  headings or labels** (no "Layman point:", no "System design:", no
  "Code change:"):
  1. **Layman** — what the change adds, in semi-technical language a
     product-minded reader gets in one read.
  2. **System design** — bird's-eye view of how it works: data flow,
     queries, failure paths — pitched at a relatively proficient technical
     person (an N+1 query, a cache TTL, a signature check — concept names,
     not line numbers).
  3. **Code change** — programmer level: which files, what changed in each,
     why, and how it behaves now.
- **Never narrate merge-conflict resolutions in the PR body** (user
  directive, 2026-10-26): the merge commit body carries those. The PR body
  says what the branch is FOR and what the code does now — the branch's
  function, then the change, in the three perspectives above.
- **Always post the follow-up comment** on the PR: "### What was done"
  numbered checklist + verified lines + exclusions. A PR without a populated
  description AND a comment is an incomplete job.
- Report: PR link, base branch, comment posted, files excluded + why.

## Environment traps (they make new sessions hallucinate — read before running anything)

Persistent file rule: **never record hard numbers or status claims here** (no
"N tests pass", "golden evals are green", "these N failures are pre-existing").
Counts and pass/fail states drift weekly; a stale claim in a persistent skill
makes the next session trust a lie. State EXPECTATIONS and METHODS only —
re-derive current truth fresh each session (`git stash` A/B, a rerun, a read
of CI). The notes below are the why, not a scoreboard:

- **Shell is PowerShell 5.1 on Windows, not bash.** No `&&` (chain with
  `; if ($?) { ... }`), no bash `head`/`grep`/`cat` semantics (use
  `Select-String`, `Get-Content`, `Get-ChildItem` or the provided tools) —
  bash-style commands "fail" in misleading ways and sessions invent broken
  fixes. `git push` progress goes to **stderr**: PowerShell renders it as a
  red `NativeCommandError` that is NOT a failure — read the real output.
  CRLF/LF warnings are noise. Paths with spaces need quotes; `>` redirection
  mangles encodings (commit-message BOM trap lives in `composio-automation` §2).
- **Python always through `uv`** — `uv run pytest ...`,
  `uv run python -m ...`, `uv run ruff check .`. Bare `python`/`pip` may be
  absent or point elsewhere (`pip` does not exist on this box). Imports break
  after a workspace change → `uv sync --all-packages --extra dev` (plain
  `uv sync` prunes the env — known trap). Prefer `uv run pytest` over bare
  `pytest`: pytest config injects the workspace pythonpath.
- **No `make` on Windows** — translate Makefile targets yourself
  (`make test-fast` → `uv run pytest tests/ -m "not integration" -q`); read
  the Makefile for canonical flags instead of guessing.
- **Pre-existing failures are NORMAL; zero is a fantasy.** The suite carries a
  standing population of environment-dependent failures (Windows file
  encodings, `.env` pointing redis at a docker hostname, drifted wiring
  tests). Expectations: (1) never treat baseline red as your regression —
  (2) before claiming "my change broke it", A/B with `git stash` and rerun:
  failure present without your change = pre-existing — (3) report failures
  as *triaged*, never hidden. Also: local host has **no tesseract** (real OCR
  A/B is CI-only) and eval cassettes with `:` in paths can't materialize on
  NTFS (known Windows gap, CI unaffected).
- **PowerShell `Set-Content` / `Get-Content -Raw` default to cp1252 on this
  box.** Editing a UTF-8 file that way leaves mojibake, and a bad paste can
  produce `await await` — both stay valid-looking, so `ruff` never flags
  them. Every PS-edited file gets a UTF-8 decode check plus a double-`await`
  scan before commit. Prefer the Edit/Write tools over PS content cmdlets.
- **Huge pytest param ids blow past Windows' per-variable limit.** A param id
  that embeds a multi-MB content repr makes pytest die when it sets
  `PYTEST_CURRENT_TEST` (`ValueError: env var longer than 32767 characters`),
  and a `pytest --tb=line` run over that module can emit multi-MB output lines
  and stall the shell. These are pre-existing artifacts, never a regression:
  cap such batches with `--tb=no`, run the whole file (single-node selection
  can also raise a `KeyError` on a table the module's fixtures expect), and
  never chase either as your change's fault.
- **Linear issue ids must be re-read from each call's response, never carried
  from memory or notes.** A remembered uuid silently resolves to a different
  ticket and moves the wrong issue — this happened twice. Read the
  `identifier` field in the mutation RESPONSE before trusting any id.

## Reporting style
- After each stage: 1-3 bullets — done, artifacts as markdown links, next
  expected cue. No filler. When blocked (Linear limit, ambiguous contact,
  conflict needing a semantic decision) → ask; don't guess and don't proceed.

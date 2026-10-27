---
name: composio-automation
description: Drive external apps through Composio — Gmail drafts/sends/attachments/contact lookup, GitHub PRs (find-then-create, branch/repo validation), Linear ticket creation/transitions, plus the local git + docx steps that surround them. Use when user says "email/draft/mail", "PR/pull request", "Linear ticket", "push and create PR", "upload attachment", or names Composio/Gmail/GitHub/Linear.
---

# Composio automation (Gmail · GitHub · Linear)

Battle-tested workflow notes from a full session: contact lookup → docx build →
upload → Gmail draft → git commit/push → GitHub PR. Follow the phases in order;
the gotchas are real failures that happened.

## 0. Universal rules (every Composio call)

1. **Search first, always.** `COMPOSIO_SEARCH_TOOLS` with a normalized English
   `use_case` (+ `known_fields` for ids/emails/names, never free text). It returns:
   - `recommended_plan_steps` + `known_pitfalls` for the exact tool — extract and follow them.
   - `primary_tool_slugs` — use these, never invent slugs.
   - Inline `input_schema` for some tools; others carry `schemaRef` → you MUST call
     `COMPOSIO_GET_TOOL_SCHEMAS` first. Never guess arguments.
   - `session.id` — **the session id changes between searches** (e.g. `noun` → `away`
     → `post`). Take the LATEST id and pass it in EVERY subsequent meta call
     (`session_id` on search/multi-execute/workbench). Stale id = validation error.
2. **Connection must be ACTIVE** before executing anything. Response shows
   `toolkit_connection_statuses`; if none active → `COMPOSIO_MANAGE_CONNECTIONS`
   (action `add`, returns auth link — show it as a markdown link, wait for user
   confirmation) → `COMPOSIO_WAIT_FOR_CONNECTIONS` → only then execute. Never run
   a tool without an active connection.
3. **Two executors:**
   - `COMPOSIO_MULTI_EXECUTE_TOOL` — batch up to 50 **logically independent** tools
     in parallel (no output→input dependencies). Sequential chains = separate calls.
   - `COMPOSIO_REMOTE_WORKBENCH` with `run_composio_tool(slug, {...})` — Python cell,
     180 s cap, state persists across cells. **Use this whenever the payload is large
     or awkward** (see gotcha G1).
4. **Pass `current_step`** (short enum) + optional `current_step_metric` in meta calls
   so long workflows stay traceable.
5. Response too big? `sync_response_to_workbench=true`, then process the file remotely
   (`COMPOSIO_REMOTE_BASH_TOOL`). Small results: keep inline.
6. **Tool returned junk/empty?** `COMPOSIO_SUBMIT_FEEDBACK` with the slug — fixes the
   corpus for everyone.

## 1. Gmail

### Contact / address lookup (do this BEFORE composing)

**Loose name search is the norm.** User gives only a first name ("draft to
prasanna", "mail nirjala") — never an email. Default rules:

- **Domain tag: `@awddel.com`** — every internal contact lives there. Treat any
  hit whose email ends in `@awddel.com` as the candidate pool; ignore external
  addresses unless the name only matches externally.
- **Step 1 — People search:** `GMAIL_SEARCH_PEOPLE` with `query = <first name>`
  (e.g. `"prasanna"`), `person_fields="emailAddresses,names"`. `query` is required
  (empty = validation error); `person_fields` is ONE comma-separated string.
  Filter results to `@awddel.com`.
- **Step 2 — people I have already mailed (best signal):** if People search is
  empty/ambiguous, go to the mailbox — `GMAIL_FETCH_EMAILS` with a name query
  (e.g. `prasanna`) and/or `to:prasanna@awddel.com`; read the actual
  `To:`/`From:` headers (`GMAIL_FETCH_MESSAGE_BY_MESSAGE_ID` if truncated).
  A name that appears in sent/correspondence headers = the real working address.
  This is the preferred tie-breaker: **the person user already mails is the target.**
- **Selection rule:** exactly one `@awddel.com` match → use it. Multiple distinct
  `@awddel.com` addresses for the same name → prefer the one seen in mail
  headers, then `metadata.primary=true`; if still ambiguous → ASK the user with
  the candidates listed. Never guess, never pick an external domain silently.
- Results often lack `emailAddresses`/`names` — handle missing keys.
- Full fallback chain: `GMAIL_SEARCH_PEOPLE` → `GMAIL_FETCH_EMAILS` (headers) →
  `GMAIL_FETCH_MESSAGE_BY_MESSAGE_ID` → `GMAIL_GET_PEOPLE`/`GMAIL_GET_CONTACTS`.
- 401/403 on People API = missing directory scopes → go to mailbox fallback,
  don't retry.
- Remember the resolved address in-session so later drafts don't re-search.

### Draft / send
- `GMAIL_CREATE_EMAIL_DRAFT`: needs ≥1 recipient and ≥1 of subject/body.
  Returns `draft_id` + `display_url` — **always show the display_url to the user
  as a markdown link**. Draft id must be used as-is by `GMAIL_SEND_DRAFT`
  (synthetic/stale ids fail).
- **Default = draft, never send**, unless user explicitly says send. User checks
  the draft first.
- Reply-in-thread: pass `thread_id` and leave `subject` EMPTY (a subject spawns a
  new thread).
- **Progress-report email body = SHORT ONLY.** Never populate the body with the
  task list / work summary — the docx attachment carries that. Body is one or
  two lines, e.g.:
  > "Respected Sir, please review my progress for the day. Detailed task list
  > is attached as a document. Best regards, Prabhakar"
  The numbered task breakdown belongs in the ATTACHED doc, not in the email.
- User may hand-edit a draft in Gmail (subject/body change, messageId changes,
  threadId kept) — when they say "send it", send AS-IS: verify recipient
  matches the intended person, do NOT re-edit content, do not "fix" their
  adjustments (including missing/changed attachments — their call).
- HTTP 429 on rapid create/send → exponential backoff (1s/2s/4s), don't hammer.
- Recipients: plain names without `@` are invalid; `Name <addr@x>` format OK.

### Attachments (3-step, learn this cold)
1. **Get the file into the sandbox.** Workbench files are remote; local Windows/macOS
   files are not visible there. Small files (< ~100 KB): read bytes locally, base64
   (`[Convert]::ToBase64String([IO.File]::ReadAllBytes($path))`), embed the b64 string
   in `code_to_execute`, `base64.b64decode(...)` → write `/mnt/files/<name>`.
2. **Upload:** call helper `upload_local_file(path)` inside the workbench cell →
   returns `s3key` (e.g. `project/.../5m76xzFMjDIa`) and `s3_url`.
3. **Attach:** `attachment: {name, mimetype, s3key}` (or a list) on the draft tool.
   Mimetype must match the real type —
   docx = `application/vnd.openxmlformats-officedocument.wordprocessingml.document`.
   Message budget: 25 MB total including base64.
4. **Clean up** local temp artifacts after (user often asks "delete the local doc").

## 2. GitHub

### PR creation = validate → dedupe → create → comment
1. Repo identity from `git remote -v` (owner/repo, strip `.git`).
2. **Dedupe first:** `GITHUB_FIND_PULL_REQUESTS` with `head_branch`,
   `base_branch`, `state="open"`, `repo="owner/repo"`. `total_count: 0` → safe.
3. `GITHUB_CREATE_A_PULL_REQUEST`: required `owner`, `repo`, `head`, `base`;
   `title` or `issue`. Same-repo head = plain branch name (owner:branch only for
   forks). Non-draft unless user says draft.
4. Persist `number` + `html_url` from the response immediately — the create
   response is HUGE (repo object and all); show the PR link as markdown.
5. **PR description is MANDATORY and must be substantial.** Never create a
   bare/one-line PR. Structure it (see §7 template): Summary, What changed
   (per-ticket/per-area subsections), Test plan, Notes/exclusions. The body is
   populated at create time from the actual diff + commit — read `git diff
   --stat` and the commit bodies before writing it.
6. **Post a PR comment after creating** (not just the description): use the
   issue-comment endpoint on the PR (`GITHUB_ADD_A_COMMENT_TO_AN_ISSUE` — PRs
   are issues; `comment_id`-less create, `issue_number` = PR number) summarizing
   *what was done* in good format — checklist or short sections, so a reviewer
   sees the narrative without scrolling the diff. Search the tool slug first if
   unsure; the PR's own `comments_url` confirms it landed.
7. Follow-up: `GITHUB_GET_A_PULL_REQUEST` (note: `mergeable` is `null` right after
   creation — re-fetch later), `GITHUB_REQUEST_REVIEWERS_FOR_A_PULL_REQUEST`.

### Commit message discipline (technical, always)
- **Subject:** `area: imperative summary` (≤72 chars), e.g.
  `OCR quality: T1-T7 hardening`. No BOM (see PowerShell trap below).
- **Body = the technical record**, one paragraph/section per logical change:
  file paths, function names, config keys, metric names, pitfall/ticket IDs,
  WHY a decision was made, what was deliberately excluded and why, test counts
  and how failures were triaged. Write it for a future agent reading `git log`.
- Do **not** ship fluffy/vague messages ("fix stuff", "updates"). If the diff is
  big, the message is longer — mirror the PR description's substance.
- Review before push: `git log --format='%h | %s' -1` for subject,
  `git log --format=%B -1` for the full body.

### Gotchas
- HTTP 422 = invalid base/head **or duplicate PR** (re-run dedupe);
  422 "No commits between base and head" = base already contains head — check
  `git log base..head` first.
- HTTP 403 = policy/permission — not retryable.
- **Base branch matters and must be quoted by the user** (e.g.
  `integration/distributor-network`, NOT `main`) — branch-protection/CI gate reasons;
  double-check before create and say the base in your confirmation.
- Always verify the branch is pushed (`git push origin <branch>`) BEFORE creating;
  PR create fails 422/404 on a missing remote head.

### Local git around the PR (PowerShell/Windows traps)
- Stage deliberately: `git add -u` (tracked) + explicit new files. **Before adding
  anything untracked, check WHY it's untracked** (`git show <last-commit>` — e.g.
  "cassette-less goldens held back until replay fixtures" / stray docs). Excluded
  files belong in the PR description as intentional exclusions.
- Commit message with a here-string: PowerShell `Out-File -Encoding utf8` writes a
  **BOM** → shows up as `﻿subject`; fix: write with
  `[IO.File]::WriteAllText($f, $msg, [Text.UTF8Encoding]::new($false))`.
  Also `$m = git log --format=%B` captures an ARRAY of lines — joining later
  flattens to one line; use `-join "`n"` on capture, never rely on round-trip.
- Verify before push: `git log --format='%h | %s' -1` + `%B` body echo.
- Amend only pre-push; `git config user.name/user.email` may warn about auto identity.
- `git push` "new branch" output goes to stderr — not an error on Windows PowerShell.

## 3. Linear

- Ticket creation can fail `USAGE_LIMIT_EXCEEDED` (seat/credit limit). Don't retry
  blindly: report it, save every resolved id (team uuid, cycle uuid, assignee uuid,
  status column uuids for Todo/In Progress) in the session notes so creation is a
  one-shot later; let the user handle the limit.
- **Guardrail:** only this agent (via Linear MCP) may transition issues. Editing the
  local planning docs (`*.md` strike-throughs/status lines) is free — do both so the
  local source of truth stays current even while Linear is blocked.

## 4. Docx generation (for attachments)

- The `docx` npm package is usually NOT preinstalled (skill claim is optimistic):
  `npm install docx --no-audit --no-fund` in the temp dir (works, ~25 s), write a
  small `make_doc.js`, `node make_doc.js "Report.docx"`.
- python-docx / pandoc often absent — don't hunt, just npm-install.
- Verify the artifact before uploading: open the zip, read `word/document.xml`,
  assert key strings exist (PowerShell:
  `System.IO.Compression.ZipFile::OpenRead` + check regex).
- Mimetype for attachment: see §1 attachments.
- After upload+draft: delete the local docx and temp build files (user usually asks).

## 5. Worked example (canonical order)

```
1. COMPOSIO_SEARCH_TOOLS [gmail: search people "prasanna"]        → session id S1
2. GMAIL_SEARCH_PEOPLE (session S1) → @awddel.com filter → GMAIL_FETCH_EMAILS tie-break
3. Build docx locally → verify zip → base64 → workbench cell writes /mnt/files/x.docx
   → upload_local_file → s3key
4. GMAIL_CREATE_EMAIL_DRAFT (recipient, subject, body, attachment) → draft_id + url
5. Delete local docx + temp files
6. git add -u + <intended new files>; git commit (technical message, BOM-safe);
   git push origin <branch>
7. COMPOSIO_SEARCH_TOOLS [github create PR]                       → session id S2 (may differ!)
8. GITHUB_FIND_PULL_REQUESTS (dedupe, session S2)
9. GITHUB_CREATE_A_PULL_REQUEST (head, base, title, POPULATED body per §7) → show PR url
10. GITHUB_ADD_A_COMMENT_TO_AN_ISSUE (issue_number = PR #) — "what was done" summary
11. Report: draft link + PR link + comment + what was excluded and why
```

## 6. Reporting style

- Show every artifact as a markdown link (Gmail draft display_url, PR html_url).
- State explicitly: draft NOT sent; base branch used; files excluded from the
  commit and why. One short bullet list — no prose padding.

## 7. PR description + comment templates (never skip either)

**The PR body** — written at create time, from the real diff/commits:

```markdown
## Summary
<2-4 sentences: what this PR does and why, name the plan/ticket source.>

## What changed
### <Area/Ticket 1>
- `path/file.py` — concrete change (config keys, metric names, function names).
### <Area/Ticket 2>
- ...

## Test plan
- command → result (pass counts, how failures were triaged: pre-existing vs new)
- lint/typecheck results

## Notes / follow-ups (not in this PR)
- deliberate exclusions + reason; open pitfalls; deferred work
```

**The PR comment** — posted right after create, same info compressed for
reviewers (`GITHUB_ADD_A_COMMENT_TO_AN_ISSUE`, `issue_number` = PR number):

```markdown
### What was done
1. **<Ticket/area>** — one line: change + where (`path:fn`).
2. **<Ticket/area>** — one line.
...
**Verified:** `<test cmd>` → N passed; `ruff` clean on touched files.
**Deliberately excluded:** <files> — <reason>.
Full detail in the PR description above.
```

Rules: description = full technical record; comment = fast reviewer digest.
Both populated, never empty/generic. Search the exact comment-tool slug first
(`COMPOSIO_SEARCH_TOOLS`, "add comment to github pull request").

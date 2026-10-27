---
name: ste-writing
description: ASD-STE100 Simplified Technical English for ALL agent prose. Load at the START of every session — no trigger word needed. Governs every explanation, summary, plan, review comment, error report, doc paragraph and final answer the agent writes. Use when user says "STE", "ASD-STE100", "simplified technical English", "write clean and concise", "clean technical output", or by default for any session that produces explanatory text.
---

# STE writing (ASD-STE100)

Write ALL agent prose in ASD-STE100 Simplified Technical English (Issue 9).
STE is a controlled language: 53 rules + a dictionary of ~900 approved words.
Purpose: text that a non-native reader (or a downstream agent) cannot misread.

Scope: chat answers, plans, code explanations, reviews, commit/PR prose,
README and doc text, comments you propose. Code identifiers, code bodies,
stack traces, quoted output and error messages stay verbatim.

Load `references/rules.md` in this skill for the full rule set and the
word-substitution table when you must check a specific construction.

## Non-negotiable rules

1. **Short sentences.** Max 20 words per instruction. Max 25 words per
   descriptive sentence. One topic per sentence.
2. **Active voice.** Subject does the action: "The worker retries the
   call." Passive only in description, and only when the actor is unknown
   or irrelevant: "The row is stored in Postgres."
3. **Verb first in instructions.** One instruction per sentence. Start a
   conditional with the condition: "If the token is stale, renew it."
4. **Allowed verb forms only:** infinitive, imperative, simple present,
   simple past, simple future (`will` + infinitive), past participle as
   adjective. No perfect tenses ("has failed" → "failed"). No continuous
   tenses ("is running" → "runs"). No `would` / `should have` stacks.
5. **No `-ing` verbs.** "The service retries the request" not "The service
   is retrying the request". `-ing` allowed only as a noun ("the
   screening", noun compound like "connection string").
6. **Plain words.** Use `start` not begin/commence/initiate, `use` not
   utilize, `check` (verb sense: use `examine`/`verify` — `check` is noun
   in STE) , `help` not assist, `need` not require, `about` not
   approximately, `end` not terminate/cease, `show` not display/demonstrate,
   `if` not in the event that. Full table in `references/rules.md`.
7. **One word, one job.** Same thing = same word every time. Never vary a
   term for style. Never use a word in a second meaning.
8. **No noun clusters over 3 nouns.** "API key rotation policy" (3) OK.
   Break longer ones with `of`/`for`/`at`: "policy for rotation of the key".
9. **No phrasal verbs** (verb + particle): "log in" → "sign in",
   "set up" → "install and configure", "pull off" → "remove".
10. **No semicolons.** Split into two sentences. Colon only to start a list.
11. **No omitted articles.** Never drop `the`/`a` to shorten a sentence.
12. **Verbs not noun-forms.** "Examine the config" not "Do an examination
    of the config". "Test the route" not "Perform testing of the route".
13. **Paragraphs:** one topic, max 6 sentences. Use a numbered list for
    sequences, a bulleted list for sets.
14. **Safety/risk first:** command first, then the risk. "Do not force-push
    `main`. The branch is protected and the push fails."
15. **No filler.** No "Sure!", "I hope this helps", "Great question",
    "basically", "essentially", "in order to" (→ "to"), "leverage" (→
    "use"), "robust"/"seamless" haze words. State the fact.

## Process for every response

1. Draft the answer.
2. Cut every sentence over 20 words into two.
3. Swap unapproved words per `references/rules.md`.
4. Convert passive and perfect constructions to active simple forms.
5. Replace prose sequences with a list when the sentence needs a list.
6. Keep code, commands, paths, identifiers and quoted text unchanged.

## Examples

Before:
> "The ingestion pipeline has been performing deduplication of the incoming
> leads in order to ensure that we are not essentially creating duplicate
> records, which would then be causing downstream issues.

After:
> "The ingestion service deduplicates incoming leads. This action prevents
> duplicate records. Duplicate records cause errors in later stages."

Before:
> "If you're basically facing a 401 error, you should probably check whether
> the token has expired, and if it has, you can renew it via the endpoint."

After:
> "If you get a 401 error, examine the token. If the token is expired, renew
> it: `POST /hub/auth/renew`."

Before (review comment):
> "Hey, maybe consider refactoring this since it's kinda hard to follow?"

After:
> "Split this function. It has 3 tasks in 1 body. Each task needs its own
> test."

## Frontend / style conflict

STE wins for prose. UI microcopy (button labels, tooltips) may keep product
terms and established UI vocabulary — those are technical names. Everything
the agent explains to the user follows STE.

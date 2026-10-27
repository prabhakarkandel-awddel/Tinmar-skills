# ASD-STE100 rule reference (paraphrased, Issue 9)

Summary of the 53 rules in 9 sections. This is a working aid, not the
standard. Official PDF: https://www.asd-ste100.org/assets/files/ASD-STE100_ISSUE9.pdf

## Section 1 — Words

- **1.1–1.4 (dictionary use):** use an approved word only in its approved
  part of speech and meaning. One word, one job, one meaning.
- **1.5 (technical nouns):** domain terms not in the dictionary are allowed
  as technical nouns if they match a listed category (products, parts,
  tools, systems, data, etc.). Use the term the reader sees in the UI,
  schema or parts list.
- **1.6 (word consistency):** same term for the same thing in the whole
  document. No synonyms for style.
- **1.7 (short words):** prefer the plain short word over the formal one.
- **1.8 (verb for the action):** use the approved verb, not a noun built
  from it. "Examine the file" not "Do an examination of the file".
- **1.9 (adjectives):** limit adjectives. Do not stack them.
- **1.10 (adverbs):** limit adverbs. Do not stack them.
- **1.11 (pronouns):** use `you`, `we`, `they`, `it` only when the noun is
  already clear. Prefer the noun over a vague pronoun.
- **1.12 (technical verbs):** domain verbs not in the dictionary are allowed
  if they match a listed category (system actions, machining, testing…).
- **1.13 (two-part verbs / phrasal verbs):** do not build new verb +
  preposition combinations. Established technical terms are exceptions.
- **1.14 (articles):** use `the`, `a`, `an` where normal English needs them.
  Never drop them to shorten text.

## Section 2 — Noun clusters

- **2.1:** max 3 nouns in a noun cluster. Break longer groups with a
  preposition: "connection settings for the database" not "database
  connection settings policy file".
- **2.2:** do not use a noun as an adjective except established technical
  cases ("access rights", "user name", "session timeout").

## Section 3 — Verbs

- **3.1 (voice):** active voice. Passive only in description, and only when
  the actor is unknown or unimportant: "The file is stored in MinIO."
- **3.2 (tenses):** allowed forms only — infinitive, imperative, simple
  present, simple past, simple future (`will`), past participle as
  adjective. Not allowed: present/past perfect ("has failed", "had run"),
  continuous ("is running"), perfect continuous, `would`/`should` stacks.
- **3.3 (commands):** imperative for instructions: "Open the config file."
- **3.4 (-ing forms):** `-ing` verb forms not allowed. `-ing` words allowed
  only as nouns or adjectives: "the rotation", "connection string".
- **3.5 (verb + noun):** do not use a verb that repeats the meaning of its
  object: "compute the calculation" → "make the calculation" or "compute".
- **3.6 (linking verbs):** `be`, `become`, `seem`, `remain` used sparingly.
  Prefer an active verb: "The test fails" not "The test is a failure".
- **3.7 (noun-verb pairs):** do not turn a verb into a noun. Covered by 1.8.

## Section 4 — Sentences

- **4.1 (one topic):** one topic per sentence.
- **4.2 (length):** max 20 words for procedural text, max 25 for descriptive
  text. Count numbers, units and identifiers as one word each.
- **4.3 (lists):** use a vertical list for 3 or more items, conditions or
  steps. Do not bury a list in prose.
- **4.4 (no omitted parts):** never remove subject, verb or article to cut
  length. Split the sentence instead.
- **4.5 (complex sentences):** do not stack more than one condition and one
  action in a sentence. Split it.

## Section 5 — Procedures (instructions)

- **5.1:** one instruction per sentence. Each sentence is a command, verb
  first.
- **5.2:** write the condition before the action: "If the build fails,
  stop the deploy."
- **5.3:** two actions allowed in one sentence only if they happen together.
- **5.4:** safety instructions follow the format in Section 7.
- **5.5:** do not write narrative around an instruction. No "you should
  probably now consider…".

## Section 6 — Descriptive writing

- **6.1:** put the most important fact first.
- **6.2:** one topic per paragraph, max 6 sentences.
- **6.3:** short sentences (max 25 words).
- **6.4:** connect ideas with a small set of link words: `and`, `but`,
  `because`, `thus`, `so`, `if`, `when`.
- **6.5:** active voice preferred; passive allowed as in 3.1.
- **6.6:** technical terms must be clear on first use. Define once, then use
  the short form.

## Section 7 — Safety instructions (warnings/cautions)

- **7.1:** start with a clear command or condition.
- **7.2:** then state the risk: injury/death = WARNING, equipment damage =
  CAUTION.
- **7.3:** be specific. "Do not edit `alembic/versions` by hand. The
  migration chain breaks." Never "be careful".

## Section 8 — Punctuation and word count

- **8.1:** no semicolons. Write two sentences.
- **8.2:** colon only to introduce a list or an explicit continuation.
- **8.3:** hyphens only inside compounds the dictionary or domain allows
  ("sign-in token", "read-only role"). No over-hyphenation.
- **8.4:** word count rules: number + unit + identifier = 1 word.

## Section 9 — Writing practices

- **9.1:** write for the reader, not the writer.
- **9.2:** keep parallel structure in lists (all imperative, all noun
  phrases).
- **9.3:** no new phrasal verbs (see 1.13).
- **9.4:** do not use figurative language, idioms or metaphors. "Move the
  needle" → "improve the metric".

## Word substitution table (common offenders)

| Do not use | Use | Note |
|---|---|---|
| utilize | use | |
| commence / begin / initiate | start | |
| terminate / cease / conclude | end / stop | |
| facilitate | help / let | |
| assist | help | |
| require (verb sense) | need | `require()` in code stays |
| perform / carry out / execute (a task) | do | code `execute` stays |
| approximately / circa | about | |
| display / demonstrate | show | UI `display` term stays |
| obtain / acquire | get | |
| purchase | buy | |
| require (demand) | need | |
| attempt / endeavour | try | |
| additional | more / extra | |
| sufficient | enough | |
| in order to | to | |
| due to the fact that | because | |
| at this point in time | now | |
| in the event that | if | |
| prior to / subsequent to | before / after | |
| in spite of / despite the fact that | although / even though | |
| with regard to / in relation to | about / for | |
| make an adjustment to | adjust | |
| give consideration to | consider | |
| come to a conclusion | conclude | |
| give an indication of | show | |
| is able to / has the ability to | can | |
| it is necessary to / you must needs | you must / must | |
| verify (if simple check fits) | check / examine | `check` is noun in STE: "do a check" |
| validate | check | unless domain verb |
| ensure | make sure | |
| leverage | use | |
| leverage / utilize in a repo of code | keep identifier, change prose | |
| robust / seamless / holistic / leverage-y haze | cut or state the fact | |
| basically / essentially / actually / just / really | cut | |
| please / kindly (in instructions) | cut | imperative is enough |
| log in (phrasal) | sign in | |
| set up (phrasal) | install / configure | |
| pull out / pull off | remove | |
| figure out | find / solve | |
| run into (a problem) | get / have | |
| put off | delay | |
| look at | examine | |
| deal with / handle (vague) | name the actual action | |
| OK / okay (in formal docs) | correct / yes | chat OK |
| via / through (choose one) | through | `via` acceptable in path-like text |
| and/or | and, or (separate) or "one or both" | |
| respectively | cut; split the sentence | |
| caveat | restriction / note | |

## Approved-word gotchas

- `check` = noun only ("do a check", "run a check"). Verb: `examine`,
  `verify`, `test`.
- `close` (verb) = shut / end a session or circuit only.
- `access` = noun. Verb: `get access to`.
- `start` / `stop` / `end` cover most machine actions.
- `will` is the only future marker. No `shall`, no `going to`.
- `must` for obligation, `can` for ability/permission, `need` for necessity.

## Self-check before you send

1. Any sentence over 20/25 words? Split it.
2. Any perfect or continuous verb? Rewrite to simple form.
3. Any passive outside a description? Rewrite active.
4. Any unapproved synonym? Swap per table.
5. Any noun cluster over 3 nouns? Break it.
6. Any semicolon? Split.
7. Any `-ing` verb form? Rewrite.
8. Any list of 3+ items in prose? Make it vertical.
9. Code and quoted text untouched? Verify.

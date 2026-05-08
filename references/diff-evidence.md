# Diff evidence: edits Levin systematically applies after the humanizer runs

This reference is built from a corpus of 26 (humanizer-output, published)
pairs across slack/github/intercom — every observable edit Levin made between
"the skill said this is the cleanest version" and "this is what I actually
shipped." Use it as the *empirical* counterpart to the heuristic Tone /
Openers / Closers / Vocabulary sections in `voice-engineering.md`,
`voice-cross-org.md`, and `voice-external.md`. When the heuristic guidance
conflicts with the evidence below, the evidence wins.

The patterns are ranked by frequency in the corpus. The Tier 1 patterns
appear in 4-5 of 5 analysis batches. Tier 2 in 2-3. Tier 3 are
single-occurrence but high-signal because they fit a coherent voice.

Across the full corpus, the *body* of the humanizer's draft is published
near-verbatim about a third of the time. The other two-thirds of edits cluster
around six themes: tool-narration leakage, sycophantic openers/closers, link
recap vs pointer, Slack-native formatting, length compression, and (rare but
costly) factual corrections.


## Tier 1 patterns

### 1. Emit only the publishable draft. Tool narration and audit blocks bleed into the output and Levin deletes them every time.

This is the single most-frequent edit in the corpus. The humanizer's response
stream interleaves:

- The actual humanized draft.
- A "What makes the below so obviously AI generated?" audit block.
- A "Changes made:" / "Final rewrite:" recap.
- `★ Insight ──────────` boxes commenting on the choices.
- Tool-use chatter ("Now let me copy that to your clipboard.", "Posting the
  PR comment now...", "Updated. The key change was...", "Done — N chars on
  your clipboard").

Levin's clipboard / paste / `gh comment` / Slack send routinely captures the
*draft + the trailing narration*. He then strips everything after the draft
before it hits the destination. The skill should solve this at the source.

**Apply this:** when producing a humanized draft, the *final* output block
must be the publishable text alone. Put the audit, the change list, and any
narration *before* the final draft block, and clearly delimit the final
block. Recommended structure:

```
**Audit (what made the input AI-shaped):** ...
**Changes made:** ...

---FINAL DRAFT START---

<the publishable text, exactly as Levin should be able to copy and paste>

---FINAL DRAFT END---
```

Or use a single fenced code block at the end with no follow-on commentary —
nothing after the closing fence except the conversation continuation. The
contract is: *anything after the final-draft delimiter is conversation, not
content*.

**Why this matters:** the skill's tool-use chatter currently leaks because
the model treats the whole response as one stream. When Levin asks for
"humanize and copy", he picks up everything after `**Final rewrite:**`
that looks like prose, including the model's commentary about its own work.
Solving this once eliminates the most-frequent edit Levin has to make.


### 2. Strip "Happy to ..." closers. Without exception.

Across the corpus, every instance of a "Happy to {pair / chat / take it to
DM / file a feature request}" closing sentence the skill produced was
deleted by Levin before publication. There were no exceptions. The pattern
also includes its variants: "I'd be glad to ...", "more than happy to ...",
"otherwise just wanted to flag", "if it'd be useful".

**Skill produced → Levin published:**

| Skill draft (cut at publish) | What Levin shipped |
|---|---|
| `Happy to chat properly if useful!` | (deleted) |
| `Happy to pair on this or walk through anything in the report.` | (deleted; only `lmk if there's a better way to hand this off...` kept) |
| `Happy to pair on a follow-up PR if it'd be useful, otherwise just wanted to flag the shape!` | (deleted entirely) |
| `Happy to take a feature request to expose this via API if it'd unblock something.` | (deleted) |

**Apply this:** delete any sentence beginning with "Happy to" or "I'd be
glad to" from drafts. Do not substitute another sycophantic closer
("let me know if you'd like me to...", "more than happy to...", etc.) —
just delete. If a real call-to-action is needed, end on `lmk` (Levin-ism,
already in the engineering variant) or a plain trailing question
(`"Re-review when you get a sec?"` is a real Levin closer that survived
verbatim).


### 3. Trust the linked artefact. Don't recap it.

When the input contains a link to an artefact (a scorecard, an investigation
report, a Google Doc, a PR), Levin's published message compresses to a 1-3
sentence pointer. The skill defaults to inclusive paraphrase — recapping the
artefact's contents in the message body. Levin defaults to exclusive
pointer — letting the link do the work.

**Skill produced → Levin published:**

| Source signal | Skill draft body | Levin's published body |
|---|---|---|
| Scorecard link present | 86 words: *"asked 7+ clarifying questions ... 0 Poor marks, 3 Greats (Breaking Down the Problem, Understanding the Problem, Data Structures). Scorecard has the full details"* | 46 words: *"Shahjalal did really well in the Problem Solving interview and the scorecard has additional details that should help support that"* |
| Self-serve PR linked | 3 paragraphs of customer-confirmed timing, controller/method, exact client-ID | *"Left a note on the [linked conversation](…) with self-serve instructions, closing!"* |

**Apply this:** if the input contains a link to an artefact, the message
body should be a 1-3 sentence pointer to the link plus the *single most
important takeaway*. Do not paraphrase the artefact. Do not volunteer
adjacent technical advice the user did not ask for.

**Important nuance — when to inline instead of link.** This pattern applies
to *casual chat* and *short replies*. It does NOT apply to high-stakes
async engineering communication where the analysis is load-bearing. The
ROT-0592 self-review (`/Users/levindixon/src/work/2026-05-06-rot-0592-investigation-self-review.md`)
identified the opposite failure: a tl;dr GitHub comment that linked to a
6,000-word investigation, and a team that re-derived the analysis from
scratch because the substance was a click away. The rule:

- *Casual reply where the link is the answer* → trust the link.
- *Decision-making async where reviewers will act on this* → inline the
  Impact / Blast Radius / Verification sections directly. Don't make
  reviewers click.

If the user describes the work as load-bearing or asks for a "thorough"
comment, default to inlining and offer the compressed version as an
alternative the user can pick.


### 4. Render Slack-native formatting from the start. Don't emit Markdown for Slack targets.

When `target_type` is Slack, the humanizer's output should use Slack mrkdwn,
not GitHub Markdown. The skill currently emits `**bold**`, `*italic*`,
unicode emoji wrapped in asterisks, GitHub short-refs, and `[label](url)`
links. Levin systematically converts all of these on the way to publishing.

**Convert at draft time, not post-hoc:**

| GitHub Markdown (wrong for Slack) | Slack mrkdwn (correct) |
|---|---|
| `*italic*` | `_italic_` |
| `**bold**` | `*bold*` (single asterisk in Slack) |
| `🟢 Not actually secrets` | `:large_green_circle: Not actually secrets` |
| `[label](https://example.com)` | `<https://example.com\|label>` |
| Bare URLs `https://example.com` | Bare URLs *only when self-describing or when there's a single link*; for 2+ links, use `<url\|label>` |
| `intercom/intercom-api-mcp-server#144` | `<https://github.com/intercom/intercom-api-mcp-server/pull/144>` (short-refs don't auto-link in Slack) |
| `- bullet` | `• bullet` |
| `Hey Steve` | `<@U022SFCQZRN\|Steve Henry>` (when the user ID is in scope) |
| `` `default_value` `` (inline code in chat) | `default_value` (Slack renders backticks as a different background; Levin treats this as visual noise for routine identifier mentions) |

**Numbered list spacing:** in Slack mrkdwn, numbered lists need a blank line
after the lead-in to render correctly. Skill should emit:

```
A few things:

1. ...
2. ...
```

not:

```
A few things:
1. ...
2. ...
```

**Backticks rule for Slack:** keep backticks around `inline code` *only when
the surrounding sentence would be ambiguous without them* (`{{key}}`,
expressions with operators, regex). Drop them around plain identifiers
where context makes it obvious. The skill currently backticks defensively;
Levin strips defensively.


### 5. Drop "Hey {name}, ..." openers for peer Slack and peer GitHub. Lead with @-mention or the verb.

For peer-eng Slack threads where the addressee is implied by the thread,
and for peer-eng GitHub comments where the action is the point, the
"Hey {name}," opener gets cut every time.

**Skill produced → Levin published:**

| Skill draft opener | Levin's published opener |
|---|---|
| `Hey Mark, nice fix on this. Apollo.io's 4 invisible connectors was a real pain and the two-PR split was the right ship.` | `<@U09FCFJSK9B\|mark.dennis> This gate is essentially asking...` |
| `Hey Steve, quick one before next week's Q2 check-ins.` | `<@U022SFCQZRN\|Steve Henry> QQ before next week's Q2 check-ins` |
| `Hey Norm, ran the 7 against the rotation tracking sheet. The UI is showing...` | `Ran the 7 secrets with \`Last updated > 1 week\` against the <docs.google.com/...\|rotation tracking sheet>.` |
| `Hey @zilleeizad / team-integration-capabilities, this is pretty far outside my area but Micheál's Slack thread caught my eye so I had Claude Code dig in.` | `This is pretty far outside my area so I had Claude dig in.` |

**Apply this:** for Slack peer-eng / peer-IC threads, drop the salutation;
either lead with the @-mention (in `<@USERID|handle>` form when a user ID
is in scope, else `@firstname`) or start with the verb. Reserve "Hey
{name}," greetings for cross-org channels and external (customer-facing)
messages.


## Tier 2 patterns

### 6. Cut self-justifying connector clauses

The skill writes connector phrases that explain how the author got involved
or pre-empt logistics decisions. Levin trims them.

| Skill produced | Levin's edit |
|---|---|
| `but Micheál's Slack thread caught my eye so I had Claude Code dig in` | (cut to `so I had Claude dig in`) |
| `Posting here in case others want the same answer, but happy to take it to DM if that's simpler.` | (cut entirely) |
| `Would help me come into our 1:1 with the same numbers you'll have.` | (cut entirely) |

**Apply this:** when the draft contains a sentence that explains *why* the
author is participating or *where* the conversation should continue, delete
it. Levin trusts readers to handle social context themselves.


### 7. "Hey Steve, quick one" → `<@U…|Name> QQ`

For incident channels and short questions, "Hey {name}, quick one" gets
swapped for `<@USERID|name> QQ ...`. "QQ" (quick question) is a Levin
voice anchor for single-question Slack messages.

**Apply this:** when the draft is a single question into a busy channel,
prepend `QQ:` (or `Q:`) and drop the salutation. If a user ID is in scope,
use the `<@USERID|name>` mention form.


### 8. Replace verbose closers with an emoji or punctuation marker

| Skill closer | Levin closer |
|---|---|
| `please poke holes before merging.` | `please poke holes 🙏` |
| `Want me to send this as a draft?` (trailing meta-prompt) | (cut) |
| `before merging` (verb-phrase qualifier) | `🙏` |
| `if it'd be useful, otherwise just wanted to flag the shape!` | (cut) |

**Apply this:** Levin's natural closer is an emoji (`🙏`), a one-word
exclamation (`closing!`), or a plain trailing question. The skill writes
verb-phrase qualifiers; swap them for the emoji or just stop.


### 9. Strip discourse-marker openers ("That said, I think,", "More importantly,", "Worth noting,")

These sneak into mid-message transitions. Levin removes them.

| Skill produced | Levin published |
|---|---|
| `That said, I think the $398K projection overstates the risk:` | `The $398K projection has some issues though.` |
| `More importantly, I have a PR open...` | `I have a PR open...` |

**Apply this:** drop "That said,", "More importantly,", "Worth noting,",
"It's worth pointing out that," — these are AI-tells the existing skill
catches in some sentences but misses on numbered-list intros and
post-pivot sentences.


### 10. Em-dashes are audience-calibrated, not banned

The current skill says "NEVER uses em dashes (`—`) or double hyphens (`--`)
as drama." Evidence shows that's *too strict for engineering PR bodies*.

| Audience | Em-dash policy |
|---|---|
| Slack chat (any variant) | Strip — replace with comma, period, parenthetical. |
| External / customer-facing | Strip. |
| Engineering PR body (dense technical clauses) | Fine. Levin keeps `"stored it in SQLite, and read it back — every MCP tool call paid for DO Request..."` |
| GitHub issue comment / PR review | Fine if the dash is doing structural work (separating clauses dense with code refs). |

**Apply this:** apply the em-dash strip rule to Slack and external
variants by default. In engineering PR bodies and dense GitHub comments,
keep em dashes that separate dense technical clauses. The dash sweep step
in the skill's Process should be variant-aware.


## Tier 3 patterns

### 11. `;` → `?` in interpretive parentheticals

When a parenthetical conveys interpretation or guess, Levin prefers a
trailing `?` over `;` — signals "I'm guessing" rather than "I'm asserting
two related things."

```
Skill: (I think that's what they actually mean; "extending" is a bit ambiguous)
Levin: (I think that's what they actually mean? "extending" is a bit ambiguous)
```


### 12. Drop release-stage acronyms (GA, EA, EAP) for cross-org Slack

Internal jargon for cross-org readers. Replace with prepositions.

```
Skill: Fin is an MCP client (MCP Connectors, GA)
Levin: Fin is an MCP client (via MCP Connectors)
```


### 13. Linkify ≥2 URLs with semantic labels and `•` bullets

Quantitative rule:

- 1 link → bare URL is fine.
- 2+ links → use `•` bullets with one-phrase labels: `• Intercom as an MCP client: <url>`.


### 14. "Less specific" doesn't mean "drop the names"

When the user asks the skill to be "less specific", the cruft to drop is
demonstration-of-knowledge details (the three exfil-repo names, the
specific permission strings). Names of *people whose tokens are in scope*
are load-bearing context for an incident channel and should be kept.

```
User: "Can we rephrase this in less specific terms and be more succinct"
Skill: outside our org's reach (personal accounts, other orgs the affected accounts belong to)
Levin: in repos that Niall's or Americo's tokens could push to (public + private personal repos, repos in other orgs they belong to)
```

Names + scope qualifier kept; demonstration-of-knowledge cruft cut.


### 15. Mandatory attribution bylines must be preserved

Different surfaces, different conventions:

- **PR descriptions:** `<sub>Generated with Claude Code</sub>` footer (managed by `developer-tools:create-pr`).
- **Automated comments / Slack:** `[~ Automated via Claude](https://github.com/intercom/claude-plugins/blob/main/plugins/base/docs/external-message-attribution.md)` byline (managed by external-message-attribution).

The humanizer must not strip these. They are not stylistic choices — they
are tooling contracts.


### 16. Single-line `<details>` in GitHub, with the report title as the summary

```
Skill:
<details>
<summary>Full investigation report</summary>

Levin:
<details><summary>Investigation: UserAuthToken blocks `legacy_secret_key_base` rotation in EU/AU</summary>
```

Single-line form, summary text repeats the actual report title.


### 17. Factual entity verification before shipping customer-facing text

Levin corrected a fabricated permission name (`Manage apps and integrations`
→ `Can install, configure and delete apps`). Humanized customer replies
that name specific permissions, products, dashboards, or settings should
include a verification step or a `[VERIFY: <name>]` marker the user can
review before sending.


## Tier 4 — substance, not style

### 18. The "AI-scraped data dump" pattern (Steve Henry feedback)

This is the highest-stakes failure mode. From Levin's manager:

> *"On the initial AI generated document, while you are encouraged to
> leverage AI in all workflows, there is an expectation that you apply
> your engineering opinion to the content. A concise document capturing
> the key data, evidence, and summarising the potential paths forward is
> more valuable than multiple pages of AI scraped data."*

In the corpus, this shows up as: skill produces a dense factual document,
Levin re-engages with engineering judgment and rewrites for argument.
Example: `9ba5f569__01` — Levin replaced the humanizer's hand-wavy
"code mode architecture change" with concrete production-vs-staging
telemetry numbers. The diff *expanded* (+12% length) because Levin added
verified facts that the humanizer's summary had elided.

**Apply this:** when the input is multi-paragraph AI-scraped content
without a clear engineering claim, the humanizer should NOT just clean
the AI tells. It should:

1. Identify the engineering claim the document is making. If there isn't
   one, surface that to the user before drafting: *"This reads as data
   dump rather than an argument. Do you want me to compress it to the
   single load-bearing claim plus the supporting evidence, or do you have
   a specific POV you want me to anchor it on?"*
2. Bias toward compression. If the input is multi-page, the output should
   be a TL;DR-with-evidence shape, not a faithful trim.
3. Refuse to ship if the work needs verifiable facts the input doesn't
   contain. Better to surface the gap than to ship plausible-but-unverified
   content.


### 19. The "Claude did most of this" disclaimer trap (ROT-0592 self-review)

Specific case: in `3dca13a4`, the user prompt asked for humility ("be
humble and forthcoming about this not being an area of expertise and
that the bulk of understanding and implementation has been left to
Claude Code"). The skill produced and Levin shipped a triple disclaimer:

> *"This is pretty far outside my area so I had Claude dig in. The bulk
> of this is Claude Code's work, not mine, so please poke holes 🙏"*

The self-review documented this as a costly framing error:

> *"The intent was humility. The effect was downgrading my own confidence
> in the artifact, giving readers permission to discount it... The
> artifact's quality should be the signal, not the disclaimer."*

**Apply this:** when the user asks the skill to be "humble" or to "credit
Claude" on rigorous engineering work, *push back briefly before drafting*:

> *"Heads up — the analysis you're shipping is rigorous and load-bearing.
> Putting 'this is mostly Claude's work' in front will downgrade reader
> trust, which is what `2026-05-06-rot-0592-investigation-self-review.md`
> documented as a costly framing error. Want me to (a) ship it as
> 'Posting an investigation, please poke holes' without the
> Claude-attribution; (b) keep your humility framing verbatim; or
> (c) attribute the methodology ('I had Claude run /diagnose against
> this — flagging that the reasoning trail is in the session') without
> downgrading the work?"*

The user can override. But the default for rigorous work is option (a) or
(c), not the verbatim humility request.


## What the skill is already getting right (don't change these)

- `lmk`, `btw`, `fwiw` — survive 100% when produced. Skill is using these well.
- PR review template (technical lede + bolded "Approving with N non-blocking suggestions" line) — published verbatim in the corpus.
- "Re-review when you get a sec?" — preserved verbatim.
- GitHub long-form comments — published nearly verbatim after stripping the surrounding ` ``` ` fence the skill wrapped them in.
- "I'm a bit in over my head on the verification side" — *kept* by Levin as load-bearing scope calibration. Distinguish this from the ROT-0592 disclaimer: scope calibration ("I can't enumerate this without console_execute") is honest-and-load-bearing; "Claude did most of this" is work-downgrading.
- No "stands as", "tapestry", "evolving landscape", "rule of three" rescues observed in the corpus. The skill is winning these fights when format is otherwise clean.


## Length / structure aggregates

Across 26 diffs:
- Median size ratio (published / skill final) = 1.00.
- For chat replies with a linked artefact: ~0.5× (Levin compresses aggressively).
- For standalone Slack messages: ~0.7-0.9× (light tightening).
- For long structured GitHub comments: ~1.0× (verbatim publish, after fence strip).
- For PR bodies: ~1.0× when the skill produces correct format; ~0× when Levin discards the draft and writes a fresh `### Why? / ### How? / ### Decisions / ### Validation` PR description (this is workflow, not edit — the humanizer wasn't the right tool for that artifact).

**Compression target by audience:**

| Variant | Target chat-reply length |
|---|---|
| engineering | 1-4 sentences for chat. Long-form is for PR / RCA / writeup, not chat. |
| cross-org | similar to engineering, plus tightened bullets. |
| external | the most aggressive compression — strip all internal markers, leave only the customer-actionable sentence. |

# Voice variant: cross-org / internal

Use this variant for Slack messages to a cross-org internal audience
(`#announce-*`, `#fin-mcp-wg`, `#ask-fin`, `#rd-services`, `#product-updates`,
partner-facing internal channels, intake threads from CS / Sales / PM). Still
internal, but the reader may not know the system, so some extra framing is warranted.

## Tone

- Direct but slightly more context-setting than peer eng chat. Explain the why
  before the what.
- Acknowledge the asker by name when they asked something, then go straight to
  substance.
- When pushing back, do it with evidence: cite the Gong call / Kibana / PR / file
  path / customer name explicitly.
- Comfortable naming customers and dollar amounts when relevant ($900K TCB,
  Back Market, Norion, Moonpig). Internal-confidential framing is fine.
- Correct misinformation gently but clearly ("worth noting: their CSM told them on
  the call this wasn't possible, which is a miscommunication we should correct").

## Sentence length and structure

- Mixed. A single-sentence acknowledgement ("Understood, thanks {name}!") can sit
  alongside a 6-10 sentence multi-paragraph technical breakdown in the same thread.
- For technical breakdowns: a one-line header sentence ("I want to flag a gap
  between what this PR delivers and what's actually blocking {customer}.") followed
  by numbered points with evidence (Gong link, code path, file ref).
- Quote the original ask with `>` before answering it.

## Formatting habits

- Numbered points with embedded code paths and links.
- Inline backticks for service/file/method names (`MergeConversation`,
  `valid_channel_combination?`, `app/lib/...`).
- Linked timestamps to Gong / Kibana / Snowflake / GitHub for evidence.
- Sparing bold. Prefer context over `**bold this**`.
- Markdown headings (`##`, `###`) only in standalone writeups (RCAs, on-call
  1-pagers), never in chat threads.

## Vocabulary used in this variant

- "Hey {name}, thanks for flagging this."
- "I want to flag a gap…", "Worth noting:", "Could you add a bit more context
  around…"
- "We can own the implementation, if you want to create a GitHub issue that details
  the requirements and timelines that would be really helpful."
- "this is top of mind for me", "I don't have a timeline yet but I'm happy to come
  back here next week", "happy to dig into"
- "in theory we could …", "we'd need to take a look at the scope of …"

## Vocabulary to avoid (on top of the Shared Voice Principles in SKILL.md)

- Marketing language ("powerful", "seamless", "groundbreaking"). The one
  `#product-updates` post that opened "Excited to share a powerful new webhook…" is
  the ceiling of marketing tone — don't drift further.
- Vague attributions ("Industry observers", "Experts argue"). Always name the
  source: "BER (Back Market's lead) explicitly stated …", "Jeff confirmed …".
- Internal jargon a non-eng reader can't parse ("RBAC", flag names) without a
  one-line gloss.
- Promises without timelines or owners.

## Openers

- "Hey {name}, thanks for flagging this."
- "Hi {name}, …"
- "I want to flag a gap between …"
- "Great question {name}, …" (used sparingly, only when it actually is one).
- "Could you add a bit more context around …"

## Closers

- "Thanks {name}!"
- A clarifying question back ("Have you thought much about how you would like this
  limitation to be communicated to customers …?").
- "We can own the implementation, if you want to create a GitHub issue …"
- Action handoff: who's doing what next.

## Examples

### Thanks-for-flagging with timeline framing
> Hey {name}, I have "Audit MCP Server latest spec (YYYY-MM-DD) adherence" on my
> todo list this week so this is top of mind for me. I don't have a timeline yet
> but I'm happy to come back here and update you with a better idea next week.

### Pushback with evidence (lede + numbered points)
> I want to flag a gap between what this PR delivers and what's actually blocking
> {customer}. I had Claude parse through the {customer} CSM sync transcript
> ({date}) and their situation is more layered than the report line item suggests:
> 1. They need merge via API — your PR addresses this.
> 2. They need cross-type merging (phone calls into tickets/conversations) — this
>    actually already works in the existing `MergeConversation` service
>    (`valid_channel_combination?` supports PhoneCall → Email/Messenger).
>
> Worth noting: their CSM told them on the call this wasn't possible, which is a
> miscommunication we should correct.

### Intake with a clarifying question
> We're able to monitor the app package associated with the server, so in theory
> we could set up these alerts today. I'll double check the existing implementation
> doesn't expose any of these. Could you add a bit more context around why we need
> to ensure these aren't exposed? That will help me determine the best suited
> method of preventing them from being accessed by the {server}.

### Direct answer to a partner question
> The ticket field in both POST /conversations/search and GET /conversations is
> stable and supported, you can safely rely on it. It was introduced in API
> version 2.9 (May 2023) and is version-gated. You may have noticed it doesn't
> appear in our public changelog, that was an oversight in how it was published,
> not a reflection of its stability.

## Length expectations

- Short ack = 1 sentence.
- Substantive answer = 1 lede + 2-4 numbered points with evidence.
- If it's longer than ~12 lines of prose, write it as a doc and link it.

## Drop-in system prompt (for downstream tools)

```
You are drafting a Slack message from me to a cross-org internal audience
(#announce-*, #fin-mcp-wg, #ask-fin, #rd-services, intake threads from CS/Sales/PM).

Voice:
- Direct but slightly more context-setting than peer eng chat. Explain the why before the what.
- Acknowledge the asker by name, then go straight to substance.
- When pushing back, do it with evidence: cite the Gong/Kibana/PR/file path/customer name explicitly.
- Name customers and stakes (e.g. "{customer} ($900K TCB, renewal {date})") when relevant.
- Correct misinformation gently ("Worth noting: …", "this is a miscommunication we should correct").

Formatting:
- Mixed length. One-line acks ("Understood, thanks {name}!") are fine alongside multi-paragraph
  technical breakdowns in the same thread.
- For technical breakdowns: a one-line lede, then numbered points (1. 2. 3.) each carrying
  one piece of evidence (a code path, a quote, a link, a customer ref).
- Inline backticks for service/file/method names (e.g. `MergeConversation`, `valid_channel_combination?`).
- Quote the original ask with `>` before answering it.
- Bare links to Gong / GitHub / Kibana / Snowflake / docs. Sparing bold. No marketing voice.
- Markdown headings only in standalone writeups, never in chat threads.

Vocabulary I use: "thanks for flagging this", "I want to flag a gap between …", "Worth noting:",
"this is top of mind for me", "I don't have a timeline yet but happy to come back here", "in theory
we could …", "we'd need to take a look at the scope of …", "happy to dig into".

Vocabulary to avoid: marketing words (powerful, seamless, groundbreaking, vibrant), vague
attributions (Industry observers, Experts argue), unexplained internal jargon, promises
without owners or timelines, sycophancy, em-dash drama.

Openers: "Hey {name}, thanks for flagging this." / "Hi {name}, …" / "I want to flag a gap
between …" / "Could you add a bit more context around …".

Closers: "Thanks {name}!" / a clarifying question back / a clean handoff naming who owns
what next ("We can own the implementation, if you want to create a GitHub issue that details
the requirements and timelines that would be really helpful.").

Length: short ack = 1 sentence. Substantive answer = 1 lede + 2–4 numbered points with evidence.
If it's longer than ~12 lines of prose, write it as a doc and link it.
```

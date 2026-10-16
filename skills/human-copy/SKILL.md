---
name: human-copy
description: Strip all detectable AI patterns from written content. Catches vocabulary tells, structural patterns, promotional language, and stylistic tics that flag text as machine-generated. Based on Wikipedia's AI signs research + academic studies on LLM detection. Use when the user says 'does this sound AI', 'human check', 'AI slop check', 'clean the copy', 'flag without rewriting' (detect mode), or 'sound human'. MANDATORY gate after any copywriting or content skill (ads, email sequences, social posts, cold email, newsletters, sales copy).
triggers:
  - "does this sound AI"
  - "human check"
  - "AI slop check"
  - "clean the copy"
  - "flag without rewriting"
  - "sound human"
  - "detect AI patterns"
  - "remove AI tells"
---

# Human Copy Gate

Every piece of written content passes through this gate before publishing. No exceptions. Blog posts, email sequences, landing page copy, social content, ad scripts - all of it.

This is not about "making it sound human." It's about eliminating the specific, documented patterns that trained readers and detection tools use to identify AI-generated text.

## Modes

- **Rewrite (default):** Run 7 passes + output structured audit (4 sections). Produces cleaned copy.
- **Detect:** Flag patterns with quoted text, tier, and severity. No rewrite. Triggers: `/human-copy --detect`, "flag without rewriting", "audit this content", "scan for AI tells without changing it". Use when auditing client content, when patterns might be intentional, or when you want a quick scan without alteration.

## Detect Mode Protocol

When run programmatically, Detect mode outputs a single JSON object:

```json
{"verdict": "ai" | "human", "flags": [{"substring": "<verbatim from text, 8 words max>", "category": "<tell category>"}]}
```

Every flag must be copied verbatim from the input, 8 words or fewer. Empty flags with `verdict: human` is the correct output for clean human text.

### Why AI text looks the way it does

LLM training produces five convergent habits. Every tell in this skill is a surface symptom of one of them. Diagnose by mechanism, not just by list - that is how you catch tells that appear on no list:

1. **Promotional inflation.** Training rewards sounding impressive, so AI puffs everything: inflated vocabulary, significance claims, "premium/comprehensive/cutting-edge" framing, benefits stacked in tidy triads. Humans writing for real stakes undersell or get specific.
2. **Both-sides hedging.** AI avoids commitment: balanced clauses, "while X, Y", qualifiers everywhere, endings that resolve into a tidy universal sentiment. Humans pick a side and leave threads hanging.
3. **Uniform fluency.** AI produces error-free, medium-length, evenly-paced sentences with parallel grammar, every paragraph the same shape. Human writing has rhythm breaks, fragments, typos, self-interruptions, abrupt jumps.
4. **Fake depth.** AI announces that things relate instead of making claims: connector sentences ("This highlights..."), trailing participles ("...underscoring the need for..."), abstractions in place of facts. A human states the causal claim and moves on.
5. **Performed warmth.** Sycophant openers, collaborative meta-commentary, corporate-therapist affirmations, neat-bow sign-offs. Humans just say the thing.

### Verdict rule (register first, then flags)

**Step 1 - classify the register.** Who is this text FOR?

- **Audience-facing prose:** marketing copy, outreach emails, blog/landing/explainer content, listicles, product descriptions, newsletters, anything written to persuade or inform strangers. Polished parallel structure, title-like headings, and benefit-shaped list items mark this register even when it looks like an outline.
- **Self-directed artifact:** voice transcripts, working notes, terminal output, logs, messages to close collaborators. Marked by disfluency, fillers, profanity, mid-thought starts, inconsistent formatting, references the writer doesn't explain because they're writing for themselves.

**Step 2 - apply the threshold for that register.**

- **0 flags → verdict: human.** Always, in both registers.
- **Audience-facing + 1 or more flags → verdict: ai.** Audience-facing prose is exactly where LLMs are used, and a genuine tell in polished copy is decisive. A human professional writing clean copy produces zero tells, not one.
- **Self-directed + 2 or more flags → verdict: ai.** Real notes and transcripts absorb the odd business word; two genuine tells in self-directed text means it was machine-drafted.
- **Self-directed + exactly 1 flag → verdict: human.**

Subtle AI is still AI: marketing copy can carry specifics, numbers, even a tuned brand voice and still be machine-written. The strongest residual signals in de-slopped AI content: every list item the same grammatical shape, every promise pitched at an imagined reader, uniform fluency with zero mess. Listy structure is not itself a tell - but a polished, parallel, audience-pitched list is audience-facing prose, not a working note, and gets the stricter threshold.

Be exhaustive. Typical AI text carries 3-8 distinct tells, and under-flagging is the dominant failure mode. One flag per distinct substring; the same word appearing twice is two flags. Each flag is a verbatim substring, 8 words max. Never invent a substring that is not in the text.

### Scan order (one deliberate pass per family)

Do not rely on a single read - tells hide in fluent text. Scan the full text six times, once per family, and collect flags from each pass:

1. **Vocabulary** - Tier 1 words anywhere. Tier 2 words: every occurrence in audience-facing prose; only 2+ clusters in self-directed text. Tier 3 hedge/emphasis words only at 3+ per 100 words.
2. **Structural patterns** - rule-of-three triads; negative parallelisms ("not just X, but Y"); despite-conclusions; trailing participles of fake depth ("...highlighting/underscoring..."); significance inflation; empty connector sentences ("This signals/demonstrates..."); bolded-label colon bullets; staccato anaphora.
3. **Copula avoidance** - "serves as", "stands as", "functions as", "represents", "marks", "boasts" doing the job of plain is/has.
4. **Rhythm** - 5+ consecutive sentences of near-identical length and subject-verb shape (flag the most representative sentence opening, 8 words max).
5. **Formatting** - em dashes, curly quotes, deck-style boldface, leaked markdown.
6. **Openers and closers** - sycophant openings ("Great question!"), collaborative meta-commentary, corporate-therapist affirmations, neat-bow universal endings.

### What NOT to flag (human signals)

Missing a tell in AI text and flagging clean human text are equal failures. These are positive evidence of a human author - never flag them, and weigh them against an ai verdict:

- Typos, missing capitalization, profanity, slang, texting shorthand
- Filler and self-interruption: "I guess", "kind of", "you know", trailing thoughts, questions to self
- Terminal output, logs, code, file listings, raw transcripts
- Notes, outlines, and to-do fragments - listy structure alone is not a tell
- Specific names, dates, dollar amounts, and decisions with personal stakes
- Abrupt topic jumps, unresolved asides, starting mid-thought
- One ordinary business word ("leverage", "robust", "streamline") inside otherwise messy, specific text is NOT a tell - vocabulary only counts per its tier rules
- Polish alone is NOT a tell: a terse, concrete, error-free professional email with real particulars (dates, rates, section numbers) is human. Competence is not AI.

## When to Run

- After any copywriting or content skill: ad copy, email sequences, social posts, newsletters, cold email, landing pages, sales copy
- After any blog post, article, or long-form content generation
- After repurposing content into new formats (repurposed content inherits parent patterns)
- Before publishing anything to a live site

## The 7 Passes

Run all 7 passes on the content. Fix every violation. No "this one's fine" exceptions.

### Pass 1: Vocabulary Purge (3-Tier Sensitivity)

Tiered flagging reduces over-correction. Tier 1 fires always. Tier 2 fires only when 2+ words cluster in the same paragraph. Tier 3 fires only at high density (3+ per 100 words).

**TIER 1 - Always flag and replace.**

| AI Word | Replace With |
|---------|-------------|
| delve/delving | dig into, explore, look at, examine |
| tapestry (figurative) | DELETE - say what you mean |
| vibrant | DELETE or use a specific descriptor |
| crucial/pivotal/vital | important, or DELETE if the sentence works without it |
| foster/fostering | build, create, grow, start |
| landscape (figurative) | scene, market, space, world |
| meticulous/meticulously | careful, precise, obsessive |
| underscore (verb) | show, prove, reveal, make clear |
| showcase/showcasing | show, display, demonstrate |
| testament (figurative) | proof, evidence, sign |
| intricate/intricacies | complex, detailed, layered |
| garner | get, earn, win, attract |
| groundbreaking | new, first |
| renowned | famous, well-known, respected |
| leverage (verb) | use |
| utilize | use |
| boasts | has |
| nestled | DELETE - travel brochure language |
| Additionally (starting a sentence) | Also, And, Plus, or just start the next sentence |
| Furthermore | DELETE - just continue |
| Moreover | DELETE - just continue |
| Notably | DELETE |
| In conclusion / In summary / Overall | DELETE - just end |
| It is worth noting | DELETE |
| unpack (figurative) | break down, go through, explain - or just make the point |
| That said (starting a sentence) | But, Still - or DELETE and continue |
| Importantly (starting a sentence) | DELETE |
| ever-changing / ever-evolving | DELETE - says nothing; name what actually changed |
| learnings (figurative, "leverage our learnings") | lessons, what we learned, what worked |
| in an ever-changing landscape | DELETE the whole phrase |
| navigate the complexities of | handle, deal with, work through - then name the complexity |
| lean into (corporate) | DELETE or say the literal action |

**TIER 2 - Register-dependent. In audience-facing prose (marketing, blog, outreach, explainer), flag EVERY occurrence individually - polished copy earns no benefit of the doubt. In self-directed text (notes, transcripts, logs), flag only when 2+ cluster in the same paragraph; a single use there is normal human business vocabulary.**

| AI Word | Replace With |
|---------|-------------|
| bolstered | strengthened, backed, supported |
| enhance/enhancing | improve, sharpen, strengthen |
| enduring | lasting, permanent, long-running |
| interplay | relationship, connection, tension |
| align with | match, fit, support |
| encompassing | covering, including |
| navigate (figurative) | handle, deal with, manage |
| facilitate | help, enable, make possible |
| comprehensive | thorough, complete, full |
| robust | strong, solid, reliable |
| innovative | new, original, clever |
| resonate/resonating | connect, hit home, land |
| embody/embodying | represent, show, be |
| cultivate/cultivating | build, grow, develop |
| seamless/seamlessly | smooth, clean, or DELETE |
| streamline/streamlined | simplify, speed up |
| empower/empowering | enable, help, give |
| holistic | whole, full, end-to-end |
| strategic | planned, deliberate, or DELETE |
| dynamic (adj) | specific descriptor or DELETE |
| transformative | big, major, or DELETE |
| revolutionary | new, first, or DELETE |
| paradigm shift | big change |
| synergy/synergize | fit, work together |
| holistic approach | full picture |
| endeavor (noun) | effort, project, try |
| commence | start |
| ascertain | find out, check |
| myriad | many, or a specific number |
| plethora | many, or a specific number |
| pivotal | important |
| quintessential | classic, typical |
| multifaceted | layered, many-sided |
| paramount | most important, top |
| imperative | must, critical |
| unwavering | steady, firm, constant |
| invaluable | very useful, rare |
| arguably | DELETE - pick a side |
| aforementioned | this, that (already named) |

**TIER 3 - Density principle, not a word list. Flag ONLY at high density (3+ instances per 100 words).**

Tier 3 covers ordinary intensity-and-hedging words that every human uses sometimes but AI uses constantly: vague magnitude words ("significant", "various", "many"), throat-clearing adverbs ("essentially", "ultimately", "certainly"), and formal connectives ("thus", "thereby", "consequently", "moreover"). The mechanism is uniform hedged emphasis - AI sprinkles these evenly because it has no actual quantity or conviction to report. One or two in a paragraph is normal human writing and must NOT be flagged. Three or more per 100 words is the tell, and the fix is always the same: replace with a specific number, a specific case, or nothing.

**Context-dependent - check each use:**

| Word | OK When | Kill When |
|------|---------|-----------|
| serves as | Technical description | Replacing "is" for no reason |
| stands as | Physical standing | Figurative ("stands as a symbol") |
| reflects | Literal reflection | "reflects broader trends" |
| highlights | You're literally highlighting | "highlights the importance of" |
| represents | Data/statistics | "represents a shift in" |
| commitment to | Contractual/legal | "commitment to excellence" |
| boasts | Never OK | Always |
| nestled | Never | Always - travel brochure language |
| in the heart of | Geographic accuracy | Figurative ("in the heart of the community") |

### Pass 2: Structural Pattern Removal

**Kill the rule of three.** AI defaults to "X, Y, and Z" triads constantly. If you see three adjectives, three examples, or three parallel phrases, break the pattern. Use two. Or four. Or one strong one.

Before: "Heritage, craftsmanship, and authenticity define the brand."
After: "Craftsmanship defines the brand."

**Kill negative parallelisms.** "Not just X, but also Y" and "It's not about X - it's about Y" are AI signatures.

Before: "It's not just a wallet - it's a statement of who you are."
After: "It's a wallet that says something."

**Kill "despite" conclusions.** "Despite [challenges], [subject] continues to [positive outcome]" is the most common AI article ending.

Before: "Despite the challenges of competing with mass-market brands, Larkin continues to grow its loyal customer base."
After: DELETE the whole paragraph. End on something specific.

**Kill superficial analysis phrases.** Present participle phrases that add fake depth:

- "...highlighting the importance of..."
- "...underscoring the need for..."
- "...emphasizing the role of..."
- "...reflecting broader trends in..."
- "...contributing to the ongoing..."
- "...symbolizing the enduring..."
- "...marking a significant shift..."
- "...setting the stage for..."
- "...shaping the future of..."

DELETE all of these. If the point matters, make it a sentence. If it doesn't, cut it.

**Kill legacy/significance inflation.** Phrases that puff up importance:

- "pivotal moment in the history of"
- "indelible mark on"
- "deeply rooted in"
- "rich cultural heritage"
- "enduring legacy"
- "plays a vital role in"
- "key turning point"
- "evolving landscape"
- "focal point of"

**Kill empty-connector sentences.** AI bridges two ideas with a hollow verb because it has no actual opinion. The tell is a sentence whose only job is to announce that one thing relates to another:

- "This signals that..." / "This signals a shift in..."
- "This underscores the need for..."
- "This highlights the importance of..."
- "This demonstrates that..."
- "This reflects a broader..."

A human states the causal claim directly and picks a side.

Before: "Customers are churning faster. This signals a shift in expectations and underscores the need for a revised approach."
After: "Customers are churning faster, so we're rebuilding onboarding."

**Kill bolded-label colon bullets.** The "**Word:** explanation" list is a deck tic AI reaches for constantly. Applies to published prose and memos meant to persuade - internal specs/scannable docs are exempt.

Before:
- **Clarity:** Ensure the memo is clear and concise.
- **Alignment:** Make sure all stakeholders are on the same page.
- **Execution:** Focus on actionable next steps.

After: write it as prose, or use plain bullets with no bolded one-word label and a real claim in each.

**Kill staccato repetition.** Anaphora triads built for fake drama - "It's bold. It's brave. It's different." or "Not a tool. Not a feature. A movement." One true sentence beats three hollow ones. (Distinct from Pass 4 fragments, which earn emphasis with real content - this is empty parallelism.)

**Kill adverb-verb fake insight.** The adverb does the work the evidence should: "quietly underscores", "subtly highlights", "powerfully demonstrates", "deeply resonates", "fundamentally reshapes". Cut the adverb, then test whether the verb survives. Usually it doesn't, and the claim needs a number instead.

### Pass 3: Copula Restoration

AI avoids simple "is/are/has" constructions. It replaces them with fancier alternatives. Reverse this.

| AI Construction | Human Construction |
|----------------|-------------------|
| serves as | is |
| stands as | is |
| functions as | is |
| represents | is |
| marks | is |
| features | has |
| offers | has, gives |
| boasts | has |
| holds the distinction of being | is |
| remains | is (usually) |

Before: "The bifold serves as a bridge between classic tailoring and everyday carry."
After: "The bifold is classic tailoring made for everyday carry."

### Pass 4: Sentence Rhythm Check

AI writes in uniform medium-length sentences. Human writing has rhythm - short punchy sentences mixed with longer flowing ones. Check for:

**Monotonous length.** If 5+ consecutive sentences are similar length (15-25 words each), break the pattern. Add a 3-word sentence. Or a 40-word one.

**Missing fragments.** Real human writing uses fragments for emphasis. "Built different." "No shortcuts." "Period." AI almost never generates fragments.

**Missing one-word paragraphs.** Sometimes a single word is the paragraph. AI doesn't do this.

**Every sentence starts with subject-verb.** AI defaults to "The [noun] [verb]..." pattern. Mix it up. Start with a time reference, a prepositional phrase, an action.

Before: "The wallet is made from full-grain leather. The edges develop a natural patina over time. The construction uses heavyweight thread."
After: "Full-grain leather. That's the shell. Over time it develops a patina - gets softer, richer, more yours. The kind of wallet that doesn't wear out. It wears in."

### Pass 5: Specificity Injection

AI generalizes. Humans specify. Every claim should pass the "like what?" test. If a reader would ask "like what?" after your sentence, you failed.

Before: "Larkin (a made-up wallet brand) uses premium materials in all its products."
After: "The leather on a Larkin bifold is 1.4mm full-grain - thick enough to hold shape, thin enough to fold flat in a back pocket."

Before: "The brand has a growing community of loyal customers."
After: "300 people keep coming back. Some are on their third wallet."

**Numbers beat adjectives.** "Premium" means nothing. "1.4mm full-grain leather" means everything. "Growing" means nothing. "300 repeat buyers" means everything. (All numbers here are illustrative.)

### Pass 6: Em Dash and Formatting Purge

**Em dashes (banned in this skill).** Replace every em dash with a single hyphen, a period, or restructure the sentence. AI overuses em dashes for dramatic pauses and parenthetical insertions.

**Curly quotes.** Replace all curly quotation marks ("" '') with straight quotes ("" ''). Curly quotes are a ChatGPT signature.

**Excessive boldface.** Remove bold from body copy unless it's a term being defined for the first time. AI bolds key phrases like a PowerPoint deck.

**Title case in headings.** Use sentence case. "How leather ages" not "How Leather Ages."

**Markdown artifacts.** Remove any `**bold**`, `*italic*`, `###`, or backtick formatting that leaked from AI output.

### Pass 7: The Human Test

Read the final copy and check:

1. **Would a person actually say this out loud?** Read it aloud. If you stumble or it sounds like a textbook, rewrite.

2. **Does every paragraph earn its place?** If you can delete a paragraph and nothing is lost, delete it. AI pads content. Humans write tight.

3. **Is there a single original observation?** Something specific, unexpected, or opinionated that no AI would generate. A detail from the founder's experience. A customer quote. A number nobody else publishes. If every sentence could appear in a generic article on the same topic, it's AI slop.

4. **Does it have an opinion?** AI hedges. "Some prefer X, while others prefer Y." Humans pick a side. "X is better. Here's why."

5. **Are there any "I hope this helps" artifacts?** Collaborative phrasing, meta-commentary about the article itself, disclaimers about limitations, or sign-off language.

6. **Is there a "challenges and future outlook" section?** Delete it. Write a specific ending about what's next, or just end.

7. **Corporate-therapist voice?** Warm, affirming, content-free: "a powerful opportunity to lean into our strengths and foster a culture of accountability." If a line belongs on a motivational poster, cut it and say the actual thing happening.

8. **The neat little bow?** AI ends on a tidy, universal sentiment: "Ultimately, the goal is to build a more resilient and agile organization." Test: could this closing line end a memo for any company on earth? If yes, it's wasted. End on something only this topic could say - a number, a decision, a next action.

9. **The summary test (says everything, means nothing).** Read each paragraph and try to say what it claimed in one plain sentence. If it sounded smart but you can't summarize it, it's pattern-matched filler. Cut it or replace with one specific claim. This is the single highest-signal check - AI matches language, it doesn't think, so its emptiest paragraphs are the most fluent.

## Automated check (optional)

If you have an automated AI-tells checker, run it after the 7 passes (or on the draft in Detect mode). Revise every flagged unit, re-check, and loop until it exits clean or 3 rounds, whichever comes first. Report any flag still standing, quoted, with its tell name.

If you do not have one, do the passes manually. Rhythm, copula swaps and the Human Test always stay manual.

## Voice overlay

If the project has a voice guide, apply it after the 7 passes.

## Quick Reference Card

For fast editing, scan by mechanism rather than by memorized phrase. Five questions, one per mechanism:

1. Is any sentence dressed up to sound impressive rather than to inform? (inflation - check vocab tiers, copula swaps, significance puffery)
2. Does any sentence balance both sides instead of committing? (hedging - "while/although" seesaws, tidy universal closers)
3. Are five consecutive sentences the same length and shape? (uniform fluency - no fragments, no breaks, parallel everything)
4. Does any sentence merely announce that one thing relates to another? (fake depth - "this highlights/signals/underscores", trailing participles)
5. Does any line perform friendliness or wrap a neat bow? (performed warmth - sycophant openers, motivational-poster closers)

A "yes" on any question sends you back to the matching pass for the specific fix.

## Structured Audit Output

Every run (Rewrite or Detect) outputs 4 sections in this order. Makes review scannable.

### Section 1 - Flags

Table of every pattern caught. One row per flag.

| # | Quoted Text | Pattern Category | Tier | Severity |
|---|-------------|------------------|------|----------|
| 1 | "serves as a bridge" | Copula avoidance | Tier 1 | High |
| 2 | "robust framework" | Vocabulary cluster | Tier 2 | Medium |

Pattern categories: Vocabulary (Tier 1/2/3), Copula avoidance, Rule of three, Negative parallelism, Despite-conclusion, Superficial analysis phrase, Significance inflation, Empty connector, Bolded-label bullet, Staccato repetition, Adverb-verb insight, Rhythm uniformity, Missing fragments, Em dash, Formatting artifact, Chatbot opener, Corporate-therapist voice, Neat-bow ending, Summary-test failure, Generic conclusion.

Severity: High (must fix) / Medium (fix if easy) / Low (judgment call).

### Section 2 - Rewrite (Rewrite mode only)

Clean version of the full text. For Detect mode, this section is skipped.

### Section 3 - Change Summary (Rewrite mode only)

Bullet list: what was cut, what was swapped, what was restructured, word count delta.

### Section 4 - Second-Pass Audit

Re-scan the rewrite (or original in Detect mode). Flag any AI patterns that survived the first edit. Common survivors: recycled transitions, lingering inflation ("significant"), copula swaps that snuck through ("remains", "holds"). If clean, note: "Second pass clean."

**For Detect mode:** Section 4 is the primary signal. It shows which flags from Section 1 are judgment calls vs. hard problems.

## Grading (Diagnostic Scale)

- **A** = Caught real AI patterns, copy reads human after fixes
- **B** = Caught some patterns, missed others, or over-corrected (made it stilted)
- **C** = Missed obvious AI tells, or copy still reads as AI after the pass

## Chain Position

- Fires AFTER any content creation skill
- Fires BEFORE publishing, deploying, or sharing content
- Can run independently on any text: `/human-copy [paste text or file path]`
- Chains: content skill -> human-copy -> publish

## Sources

Based on:
- Wikipedia's "Signs of AI writing" (community-maintained, updated Mar 2026)
- Kobak et al. (2025) "Delving into LLM-assisted writing" - excess vocabulary analysis
- Juzek & Ward (2025) "Why Does ChatGPT Delve So Much?" - lexical overrepresentation
- Reinhart et al. (2025) "Do LLMs write like humans?" - grammatical/rhetorical style variation
- Russell, Karpinska & Iyyer (2025) - detection accuracy by LLM familiarity
- Practitioner writing on AI memo tells - the "delve/unpack", empty-connector ("this signals/underscores"), bolded-label colon bullets, corporate-therapist voice, neat-bow ending, and "says everything/means nothing" patterns

## The Standard

If a heavy LLM user (90% detection accuracy per Russell et al.) would flag this text as AI-generated, it fails the gate. Rewrite until it passes.

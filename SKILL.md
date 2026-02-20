---
name: flashcard
description: Converts system design tutorials, articles, and walkthroughs into interview-ready flashcards optimized for Quizlet import. Produces a bounded set of high-quality cards focused on reasoning, trade-offs, and patterns.
triggers:
  - /flashcard
  - flashcard
  - create flashcards from
  - generate flashcards
  - make flashcards
allowed-tools:
  - WebFetch
  - Write
  - Read
  - Bash
  - mcp__playwright__browser_navigate
  - mcp__playwright__browser_snapshot
  - mcp__playwright__browser_press_key
  - mcp__playwright__browser_close
---

# Flashcard Skill

A Claude Code skill that transforms system design tutorials into interview-ready flashcards.

## Overview

This skill takes a URL to any system design article, tutorial, or walkthrough and produces a focused set of flashcards optimized for interview preparation. The output is formatted for direct import into Quizlet.

**What this skill does:**
- Fetches content from a provided URL (public or paywalled)
- Analyzes content structure and conceptual density
- Extracts high-value system design concepts
- Generates reasoning-focused flashcard questions
- Outputs tab-separated format ready for Quizlet bulk import

**What this skill does NOT do:**
- Produce summaries or synopses
- Create exhaustive study guides
- Generate beginner-level explanations
- Reproduce step-by-step tutorials

## Invocation

```
/flashcard <URL>
/flashcard <URL> --browser    (force Playwright for paywalled content)
/flashcard <URL> --no-save    (skip saving to disk)
/flashcard <URL> --format tsv (save TSV only)
/flashcard <URL> --format md  (save Markdown only)
/flashcard <URL> --save       (force save even if auto_save is off)
```

**Examples:**
- `/flashcard https://systemdesignschool.io/problems/google-doc/solution`
- `create flashcards from https://blog.bytebytego.com/p/designing-a-url-shortener`
- `make flashcards https://medium.com/@article --browser`
- `/flashcard https://example.com/article --format tsv --no-save`

## Processing Phases

The skill processes content through five phases:

1. **Content Fetching** → Retrieve and extract article content
2. **Content Analysis** → Classify tier and identify concepts
3. **Card Generation** → Create interview-focused Q&A pairs
4. **Output Formatting** → Produce Quizlet-compatible TSV
5. **Save to Disk** → Persist flashcards to XDG data directory

---

# Phase 1: Content Fetching

## Step 1.1: Extract URL from User Input

Parse the user's message to extract the URL:
- Look for HTTP/HTTPS URLs in the input
- Check for `--browser` flag indicating forced Playwright usage
- Validate URL format before proceeding

## Step 1.2: Attempt WebFetch First

For public content, use WebFetch as the primary fetch method:

```
WebFetch with prompt:
"Extract the full article content from this page. Preserve:
- All headings and section structure
- All paragraph text
- List items and bullet points
- Alt text from images/diagrams
- Any code examples or technical diagrams descriptions

Return the content maintaining semantic structure (headings, lists, paragraphs)."
```

## Step 1.3: Handle Fetch Errors

If WebFetch fails or returns insufficient content:

**Error: 404/403**
→ "Unable to access the URL. Please verify the link is correct and publicly accessible."

**Error: Timeout**
→ "The page took too long to load. Try again or use `--browser` flag for complex pages."

**Error: Paywall Detected**
Look for indicators: "subscribe", "sign in to read", "premium content", login forms
→ "This content appears to be behind a paywall. Would you like me to try accessing it via browser? (Your browser session may have access)"

## Step 1.4: Playwright Fallback for Paywalled Content

When `--browser` flag is used OR paywall detected and user confirms:

1. **Navigate to URL:**
   ```
   mcp__playwright__browser_navigate to the target URL
   ```

2. **Take Initial Snapshot:**
   ```
   mcp__playwright__browser_snapshot to capture current state
   ```

3. **Scroll and Capture Full Content:**
   ```
   REPEAT:
     - Press End key to scroll: mcp__playwright__browser_press_key("End")
     - Wait 1 second
     - Take snapshot
     - Compare content - if no new content, stop scrolling
   ```

4. **Extract Content from Snapshots:**
   - Combine all snapshot text content
   - Preserve heading hierarchy
   - Extract diagram descriptions and alt text

5. **Close Browser:**
   ```
   mcp__playwright__browser_close
   ```

**Browser Error Handling:**
- If "Browser already in use": Inform user to close other browser sessions
- If session expired: Prompt user to re-authenticate in the browser window

---

# Phase 2: Content Analysis

## Step 2.1: Classify Content Tier

Determine the content tier based on multiple signals:

| Tier | Word Count | Section Count | Diagrams | Target Cards |
|------|------------|---------------|----------|--------------|
| Short | < 2,000 | 1-3 | 0-1 | 8-12 |
| Medium | 2,000-5,000 | 4-6 | 2-3 | 15-18 |
| Deep | > 5,000 | 7+ | 4+ | 20-25 |

**Secondary Signals:**
- Multiple subsystems described → lean toward higher tier
- Detailed trade-off discussions → increase card count
- Step-by-step implementation focus → decrease card count (less conceptual)
- Multiple architectural alternatives compared → higher tier

**Hard Cap:** Never exceed 25 flashcards regardless of content length.

## Step 2.2: Extract Concepts with Priority Scoring

Score each potential concept using this matrix:

| Concept Type | Priority Score | Always Include? |
|--------------|----------------|-----------------|
| Problem framing / requirements | +3 | Yes |
| Core architecture decisions | +3 | Yes |
| Trade-offs with reasoning | +3 | Yes |
| Failure modes and handling | +3 | Yes |
| Scaling strategies | +3 | Yes |
| Consistency/concurrency patterns | +2 | Yes |
| Reusable design patterns | +2 | Yes |
| Interview traps/misconceptions | +2 | When present |
| Component deep-dives | +1 | If space allows |
| Implementation details | -2 | Exclude |
| Step-by-step procedures | -2 | Exclude |
| System-specific trivia | -1 | Exclude |

**Pattern Recognition:**
Identify and flag these common patterns for inclusion:
- Caching strategies (write-through, write-back, cache-aside)
- Sharding/partitioning approaches
- Consistency models (strong, eventual, causal)
- Message queue patterns (pub/sub, work queues)
- Rate limiting strategies
- Load balancing methods
- Data replication approaches

## Step 2.3: Generalize System-Specific Concepts

Transform specific examples into transferable principles:

| System-Specific | Generalized |
|-----------------|-------------|
| "Google Docs uses OT" | "Real-time collaborative editing systems using Operational Transformation" |
| "Twitter's fanout-on-write" | "Write-heavy social feeds using fanout-on-write pattern" |
| "Uber's geospatial indexing" | "Location-based services with geospatial indexing" |

## Step 2.4: Select Analogy Candidates

At least 20% of cards must be analogy-based. Prioritize:

**Best Candidates for Analogies:**
- Distributed systems concepts (consensus, replication)
- Consistency models (eventual consistency, CAP theorem)
- Concurrency patterns (locks, optimistic concurrency)
- Abstract networking concepts
- Counterintuitive behaviors

**Preferred Analogy Domains:**
- Libraries (lending, reservations, catalogs)
- Restaurants (orders, kitchens, seating)
- Traffic systems (signals, flow, congestion)
- Postal/delivery systems
- Banking/transactions

---

# Phase 3: Flashcard Generation

## Step 3.1: Formulate Questions

Use reasoning-focused question stems:

| Question Type | Stem Template | Use For |
|---------------|---------------|---------|
| Reasoning | "Why does [system] use [approach]?" | Architecture decisions |
| Trade-off | "What's the trade-off of [choice] vs [alternative]?" | Design choices |
| Failure | "What breaks if [component] fails?" | Failure modes |
| Scaling | "How does [system] handle [scale challenge]?" | Scaling strategies |
| Analogy | "How would you explain [concept] using an analogy?" | Complex concepts |
| Pattern | "What pattern does [approach] represent?" | Design patterns |

**Question Rules:**
- ONE clear question per card (no compound questions)
- Avoid "What is X?" - prefer "Why is X needed?" or "How does X work?"
- Questions should test understanding, not recall
- Include context when the concept requires it

## Step 3.2: Generate Answers

**Answer Constraints:**
- Answerable verbally in ~20 seconds
- Maximum 3-5 key points if multi-part
- Use inline numbering: "1) point one; 2) point two; 3) point three"
- Self-contained without needing the original article
- Clear, direct language (no jargon or hedging)

**Answer Structure by Question Type:**

**Trade-off Questions:**
"[Choice A] provides [benefit] but sacrifices [cost]. [Choice B] offers [alternative benefit] at the cost of [alternative cost]. Choose [A] when [condition], [B] when [other condition]."

**Failure Questions:**
"If [component] fails: 1) [immediate impact]; 2) [cascading effect]; 3) [mitigation strategy]."

**Analogy Questions:**
"[Concept] is like [everyday analogy]. Just as [analogy detail], [technical concept] works by [technical detail]."

## Step 3.3: Ensure Category Balance

Distribute cards across categories with no single category exceeding 40%:

| Category | Target % | Examples |
|----------|----------|----------|
| Problem Statement | 10-15% | Requirements, constraints, goals |
| Architecture | 15-25% | High-level design, component overview |
| Components | 15-20% | Specific services, databases, queues |
| Trade-offs | 20-25% | Design decisions, alternatives |
| Failure Handling | 10-15% | Error scenarios, recovery |
| Scaling | 10-15% | Growth strategies, bottlenecks |

**Balance Rules:**
- If content doesn't support a category, skip it (don't force cards)
- Reflect the article's natural emphasis proportionally
- Ensure at least 20% are analogy cards (spread across categories)

## Step 3.4: Enforce Non-Goal Boundaries

**DO NOT generate cards that:**
- Summarize the article
- Reproduce step-by-step procedures
- Explain basic concepts (assume system design familiarity)
- Cover implementation minutiae

**If user requests out-of-scope content:**
→ "This skill is optimized for interview-ready flashcards, not [summaries/tutorials/comprehensive notes]. The flashcards focus on reasoning, trade-offs, and patterns that commonly appear in interviews."

---

# Phase 4: Output Formatting

## Step 4.1: Generate Summary Header

Before the flashcard block, output:

```
## Flashcard Summary

**Source:** [Article title or URL]
**Tier:** [Short/Medium/Deep]
**Total Cards:** [count]

**Category Distribution:**
- Problem Statement: [n] cards
- Architecture: [n] cards
- Components: [n] cards
- Trade-offs: [n] cards
- Failure Handling: [n] cards
- Scaling: [n] cards

**Analogy Cards:** [n] ([percentage]%)
```

## Step 4.2: Format Flashcards for Quizlet Import

Output flashcards in tab-separated format between clear delimiters:

```
---BEGIN FLASHCARDS---
Why does [system] use [approach]?	[Answer with reasoning, trade-offs explained]
What's the trade-off of [X] vs [Y]?	[Comparison: X provides A but costs B; Y provides C but costs D]
How would you explain [concept] using an analogy?	[Concept] is like [analogy]. Just as [analogy detail], [technical mapping].
What breaks if [component] fails?	1) [Impact 1]; 2) [Impact 2]; 3) [Mitigation]
---END FLASHCARDS---
```

**Format Rules:**
- One card per line
- Tab character (`\t`) separates question and answer
- No tabs within question or answer text
- No markdown formatting within the flashcard block
- Multi-point answers use semicolons or inline numbering (no newlines)
- No blank lines within the flashcard block

## Step 4.3: Progress Updates

During processing, provide brief status updates:

```
Fetching content from [URL]...
Analyzing content structure (detected: [tier] tier, ~[word count] words)...
Extracting concepts and generating flashcards...
Formatting [n] flashcards for Quizlet import...
```

---

# Phase 5: Save to Disk

After outputting flashcards to the conversation, persist them to the XDG data directory.

## Step 5.1: Check Save Behavior

Determine whether to save based on (in priority order):
1. `--no-save` flag → skip saving entirely
2. `--save` flag → force save regardless of config
3. `--format tsv|md|both` → override output format
4. Config `output.auto_save` (default: `true`) → save automatically
5. Config `output.format` (default: `"both"`) → TSV + Markdown

Read config from `~/.claude/skills/flashcard-skill/config/defaults.json`.

## Step 5.2: Derive Topic Slug

Generate a topic slug from the article content:
- Lowercase, hyphenated, 3-5 words
- Derived from the article's primary subject matter
- Examples: `client-server-communication`, `caching-strategies`, `rate-limiting-patterns`

## Step 5.3: Create Directory

```bash
mkdir -p ~/.local/share/flashcard-skill/cards/{topic-slug}
```

## Step 5.4: Write Files

**Collision detection:** If `{date}.tsv` or `{date}.md` already exists, append a sequence number: `{date}-2.tsv`, `{date}-3.tsv`, etc.

**TSV format:** Same as Phase 4 output — one card per line, tab-separated question and answer. No header row (Quizlet-compatible).

**Markdown format:**
```markdown
---
source: "Article Title"
tier: medium
card_count: 16
generated: 2026-02-20
topic: topic-slug
---
# Flashcards: Article Title

## Card 1 — Category
**Q:** Question text
**A:** Answer text

---

## Card 2 — Category
**Q:** Question text
**A:** Answer text
```

## Step 5.5: Update Index

Append an entry to `~/.local/share/flashcard-skill/index.json`:

```json
{
  "topic": "topic-slug",
  "source": "Article Title",
  "tier": "deep",
  "card_count": 23,
  "generated": "2026-02-20",
  "files": {
    "tsv": "cards/topic-slug/2026-02-20.tsv",
    "markdown": "cards/topic-slug/2026-02-20.md"
  }
}
```

If `index.json` doesn't exist, create it as an array with this entry. If it exists, read, append, and write back.

## Step 5.6: Confirm to User

After saving, display:

```
Saved to disk:
  TSV: ~/.local/share/flashcard-skill/cards/{topic-slug}/{date}.tsv
  MD:  ~/.local/share/flashcard-skill/cards/{topic-slug}/{date}.md
  Index updated (total: N sets)
```

---

# Quick Reference

## Card Count by Tier
- **Short** (< 2K words): 8-12 cards
- **Medium** (2-5K words): 15-18 cards
- **Deep** (> 5K words): 20-25 cards
- **Maximum**: 25 cards (hard cap)

## Required Ratios
- **Analogy cards**: Minimum 20%
- **Single category max**: 40%

## Question Stems
- Why does... / Why is...
- What's the trade-off of...
- What breaks if...
- How does [system] handle...
- How would you explain [concept] using an analogy?
- What pattern does [approach] represent?

## Output Format
- Tab-separated (question\tanswer)
- Delimited by `---BEGIN FLASHCARDS---` / `---END FLASHCARDS---`
- No markdown within flashcard content

## Storage
- **Data dir:** `~/.local/share/flashcard-skill/`
- **Cards:** `cards/{topic-slug}/{date}.tsv` + `.md`
- **Index:** `index.json` (manifest of all saved sets)
- **Config:** `~/.claude/skills/flashcard-skill/config/defaults.json`

## CLI Flags
| Flag | Effect |
|------|--------|
| `--browser` | Force Playwright for paywalled content |
| `--no-save` | Skip saving to disk |
| `--save` | Force save (overrides config) |
| `--format tsv` | Save TSV only |
| `--format md` | Save Markdown only |
| `--format both` | Save both (default) |

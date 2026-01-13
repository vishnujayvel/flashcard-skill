# Flashcard Skill for Claude Code

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://claude.ai/claude-code)

A Claude Code skill that transforms tutorials, articles, and educational content into interview-ready flashcards optimized for Quizlet import.

## What It Does

- Fetches content from any URL (public or paywalled via Playwright)
- Analyzes content structure and conceptual density
- Extracts high-value concepts across any domain
- Generates reasoning-focused flashcard questions
- Outputs tab-separated format ready for Quizlet bulk import

## What It Does NOT Do

- Produce summaries or synopses
- Create exhaustive study guides
- Generate beginner-level explanations
- Reproduce step-by-step tutorials

---

## Requirements & Dependencies

### Required
- **Claude Code CLI** - The skill runs within Claude Code

### Optional (for Paywalled Content)
- **Playwright MCP Server** - Only needed if you want to access content behind paywalls (Medium, paid newsletters, etc.)

### Experience Without Playwright

| Scenario | Without Playwright | With Playwright |
|----------|-------------------|-----------------|
| Public URLs (Wikipedia, free blogs) | Works perfectly | Works perfectly |
| Paywalled content (Medium, Substack) | Fails with error message | Works if you're logged in via browser |
| JavaScript-heavy sites | May get incomplete content | Full content extraction |

**Bottom line:** Start without Playwright. Add it only if you hit paywalled content you need.

---

## Installation

### Step 1: Install the Skill

**Global Installation (Recommended):**
```bash
# Clone the repo
git clone https://github.com/vishnujayvel/flashcard-skill.git

# Copy to global skills directory
mkdir -p ~/.claude/skills/flashcard-skill
cp flashcard-skill/SKILL.md ~/.claude/skills/flashcard-skill/
```

**Project-Level Installation:**
```bash
mkdir -p .claude/skills/flashcard-skill
cp flashcard-skill/SKILL.md .claude/skills/flashcard-skill/
```

### Step 2: Verify Installation

Start a new Claude Code session and run:
```
/flashcard https://en.wikipedia.org/wiki/Binary_search_algorithm
```

You should see:
1. "Fetching content from..." status message
2. Flashcard summary with tier and card count
3. Tab-separated flashcards between delimiters

### Step 3: (Optional) Add Playwright for Paywalled Content

If you need to access paywalled content:

1. Install Playwright MCP server (see [Playwright MCP docs](https://github.com/anthropics/claude-code/blob/main/docs/mcp.md))
2. Use the `--browser` flag: `/flashcard <URL> --browser`

---

## Usage

```
/flashcard <URL>                    # Use WebFetch (public content)
/flashcard <URL> --browser          # Force Playwright (paywalled/JS-heavy)
```

### Examples

```
/flashcard https://en.wikipedia.org/wiki/CAP_theorem
/flashcard https://blog.bytebytego.com/p/designing-a-url-shortener
/flashcard https://medium.com/@article --browser
```

---

## Output Format

The skill produces:

1. **Summary Header** with source, tier classification, and category distribution
2. **Tab-Separated Flashcards** delimited by `---BEGIN FLASHCARDS---` / `---END FLASHCARDS---`

### Importing to Quizlet

1. Go to [quizlet.com/create-set](https://quizlet.com/create-set)
2. Click "Import from Word, Excel, Google Docs, etc."
3. Copy everything between the flashcard delimiters
4. Set delimiter: **Tab** between term and definition
5. Set rows: **New line** separates cards
6. Click "Import"

---

## Card Generation Rules

| Tier | Word Count | Target Cards |
|------|------------|--------------|
| Short | < 2,000 | 8-12 |
| Medium | 2,000-5,000 | 15-18 |
| Deep | > 5,000 | 20-25 |

**Hard cap:** 25 flashcards maximum

**Required ratios:**
- Minimum 20% analogy-based cards
- No single category exceeds 40%

## Question Types

- **Reasoning:** "Why does [system] use [approach]?"
- **Trade-off:** "What's the trade-off of [X] vs [Y]?"
- **Failure:** "What breaks if [component] fails?"
- **Scaling:** "How does [system] handle [scale challenge]?"
- **Analogy:** "How would you explain [concept] using an analogy?"
- **Pattern:** "What pattern does [approach] represent?"

---

## Best Use Cases

| Domain | Quality | Notes |
|--------|---------|-------|
| System design tutorials | Excellent | Core use case, optimized for this |
| Programming concepts | Excellent | Algorithms, data structures, patterns |
| Technical deep-dives | Good | Architecture, infrastructure |
| Project management | Good | Methodologies, frameworks |
| Academic CS content | Good | Wikipedia, textbook chapters |
| Beginner tutorials | Fair | Skill focuses on "why" not "how-to" |
| Non-technical content | Variable | May produce thin results |

---

## Limitations

- **Hard cap of 25 cards** - Cannot generate more regardless of content length
- **Reasoning-focused** - Won't produce memorization cards (dates, names, syntax)
- **English only** - Not tested with non-English content
- **No customization** - Cannot adjust card count, categories, or analogy percentage
- **Synthetic examples** - Example outputs are demonstrations, not from real URLs

---

## Troubleshooting

### "Unable to access the URL"
- Verify the URL is correct and publicly accessible
- Try opening the URL in your browser first
- If paywalled, use `--browser` flag with Playwright installed

### Incomplete or thin content
- The page may be JavaScript-heavy; try `--browser` flag
- Very short articles (< 500 words) produce fewer, lower-quality cards
- Consider finding a more comprehensive source

### Playwright errors
- "Browser already in use" - Close other Claude Code browser sessions
- "Session expired" - Re-authenticate in the Playwright browser window
- Playwright not installed - Follow setup guide or use public URLs only

### Cards don't import to Quizlet
- Ensure you copied only content between the delimiters
- Check for stray newlines or formatting
- Try importing a smaller batch first

---

## Example Outputs

See the `examples/` directory for sample outputs across different domains:

| Example | Audience | Topic |
|---------|----------|-------|
| [students-database-indexing.txt](examples/students-database-indexing.txt) | CS Students | Database indexing fundamentals |
| [tech-workers-rate-limiter.txt](examples/tech-workers-rate-limiter.txt) | Software Engineers | Rate limiter system design |
| [professionals-agile-methodology.txt](examples/professionals-agile-methodology.txt) | PMs / Team Leads | Agile/Scrum methodology |

**Note:** These are synthetic examples demonstrating output format. Real outputs will vary based on source content.

---

## License

MIT License - see [LICENSE](LICENSE) for details.

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## Acknowledgments

Built with [Claude Code](https://claude.ai/claude-code) by Anthropic.

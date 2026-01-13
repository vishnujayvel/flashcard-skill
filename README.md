# Flashcard Skill for Claude Code

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://claude.ai/claude-code)

A Claude Code skill that transforms system design tutorials, articles, and walkthroughs into interview-ready flashcards optimized for Quizlet import.

## What It Does

- Fetches content from any URL (public or paywalled via Playwright)
- Analyzes content structure and conceptual density
- Extracts high-value system design concepts
- Generates reasoning-focused flashcard questions
- Outputs tab-separated format ready for Quizlet bulk import

## What It Does NOT Do

- Produce summaries or synopses
- Create exhaustive study guides
- Generate beginner-level explanations
- Reproduce step-by-step tutorials

## Installation

### Option 1: Global Installation (Recommended)

Copy `SKILL.md` to your global Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/flashcard-skill
cp SKILL.md ~/.claude/skills/flashcard-skill/
```

The skill will now be available in all your Claude Code sessions.

### Option 2: Project-Level Installation

Copy `SKILL.md` to your project's `.claude/skills/` directory:

```bash
mkdir -p .claude/skills/flashcard-skill
cp SKILL.md .claude/skills/flashcard-skill/
```

## Usage

```
/flashcard <URL>
/flashcard <URL> --browser  (force Playwright for paywalled content)
```

### Examples

```
/flashcard https://systemdesignschool.io/problems/google-doc/solution
create flashcards from https://blog.bytebytego.com/p/designing-a-url-shortener
make flashcards https://medium.com/@article --browser
```

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

## Requirements

- [Claude Code](https://claude.ai/claude-code) CLI
- **Optional:** Playwright MCP server (for accessing paywalled content)

## Example Output

See [examples/google-docs.txt](examples/google-docs.txt) for a complete example output from a Google Docs system design tutorial.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

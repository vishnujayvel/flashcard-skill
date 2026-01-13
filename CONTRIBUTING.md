# Contributing to Flashcard Skill

Thank you for your interest in contributing to the Flashcard Skill for Claude Code!

## How to Contribute

### Reporting Issues

If you encounter a bug or have a feature request:

1. Check existing [issues](../../issues) to avoid duplicates
2. Open a new issue with:
   - Clear description of the problem or suggestion
   - Steps to reproduce (for bugs)
   - Example URLs that demonstrate the issue
   - Expected vs actual behavior

### Suggesting Improvements

The skill is implemented as a single `SKILL.md` file containing structured instructions. Improvements might include:

- New question types or stems
- Better category balance rules
- Additional analogy domains
- Improved error handling
- Support for new content types

### Submitting Changes

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-improvement`)
3. Make your changes to `SKILL.md`
4. Test with several different URLs to ensure quality
5. Commit your changes (`git commit -m 'Add: description of change'`)
6. Push to your fork (`git push origin feature/my-improvement`)
7. Open a Pull Request

### Testing Your Changes

Before submitting, test with at least 3 different URLs:

1. A short article (< 2,000 words)
2. A medium article (2,000-5,000 words)
3. A deep dive (> 5,000 words)

Verify:
- Card count matches tier expectations
- At least 20% analogy cards
- No single category exceeds 40%
- Output imports correctly to Quizlet

## Code of Conduct

- Be respectful and constructive
- Focus on the content, not the person
- Welcome newcomers and help them contribute

## Questions?

Open an issue with the "question" label if you need help or clarification.

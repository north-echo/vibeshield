# Contributing to VibeShield

Thank you for your interest in making AI-generated code more secure.

## How to Contribute

### Reporting a New Vulnerability Pattern

If you've identified a recurring security flaw in AI-generated code that isn't covered by the current ruleset:

1. Open an issue with the title: `[New Pattern] Brief description`
2. Include:
   - Description of the vulnerability
   - Which AI coding tools produce it (with examples if possible)
   - Supporting evidence (CVEs, advisories, blog posts, or your own test results)
   - Suggested rule text (imperative, specific)

### Proposing a Rule Change

1. Fork the repo and create a branch from `main`
2. Edit `core/vibeshield-rules.md` (the canonical source of truth)
3. Update the corresponding tool-specific files in `rules/` to match
4. If adding a new V-ID, add its entry to `evidence/vulnerability-map.md`
5. Submit a pull request with a clear description of the change and its rationale

### Adding a Tool Adapter

To add support for a new AI coding tool:

1. Create the appropriate rules file in `rules/` formatted for that tool's conventions
2. Derive all content from `core/vibeshield-rules.md` — adapt formatting only, do not add or remove rules
3. Document any tool-specific formatting requirements in your PR

### Adding a Stack Supplement

Stack-specific supplements live in `stacks/`. These add framework-specific rules that complement the core ruleset.

1. Name the file descriptively (e.g., `rails.md`, `spring-boot.md`)
2. Focus on security patterns specific to that stack — don't duplicate core rules
3. Include evidence for each rule where possible

## Guidelines

- **Keep it concise.** Context window budget is real. Every word competes with the user's code.
- **Be imperative.** "Never do X" not "Consider avoiding X."
- **Be specific.** "Use bcrypt or argon2" not "use strong hashing."
- **Cite evidence.** Every rule should trace to a documented AI vulnerability pattern.
- **Test your changes.** Run prompts from `tests/test-prompts.md` to verify rules trigger correctly.

## Code of Conduct

Be respectful, constructive, and focused on making AI-generated code safer for everyone.

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0.

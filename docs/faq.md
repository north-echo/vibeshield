# Frequently Asked Questions

## Does this actually work?

Context rules measurably influence AI code generation. VibeShield targets specific, documented AI failure patterns where a clear instruction consistently changes output. It's probabilistic, not deterministic — it significantly reduces but does not eliminate vulnerabilities.

## Do I still need SAST/security testing?

Yes. VibeShield is one layer in defense-in-depth. It reduces the volume of vulnerabilities generated, but it doesn't guarantee zero vulnerabilities.

## Won't this eat up my context window?

The core rules file is under 4,000 tokens. That's roughly 5-10% of a typical context window. The security benefit outweighs the token cost. Stack supplements are optional and add ~1,000-2,000 tokens each.

## Which AI coding tool does this work best with?

Tools that strongly respect their config files (Claude Code with CLAUDE.md, Cursor with .cursorrules) show the highest compliance. Effectiveness varies by tool. We're documenting compliance rates as we test.

## Can I customize the rules?

Yes. The rules files are plain text. Add, remove, or modify rules to match your project's needs. For team use, commit the file to your repo.

## How is this different from just telling the AI "write secure code"?

"Write secure code" is vague. VibeShield gives specific, imperative instructions targeting documented failure patterns. "Use bcrypt with cost factor 12+" is actionable; "use strong hashing" is not.

## How do I use stack supplements?

Copy the relevant stack file alongside your tool's rules file. For Claude Code, you can append the stack supplement to your CLAUDE.md or place it in a subdirectory that Claude Code reads. For Cursor, append to .cursorrules.

## How often is this updated?

The project tracks AI-linked CVEs via the Vibe Security Radar and updates rules when new vulnerability patterns emerge. Check the CHANGELOG for update history.

## Can I contribute?

Yes. See CONTRIBUTING.md. We especially welcome: new vulnerability pattern reports, new tool adapter formats, new stack supplements, and effectiveness test results.

## Is this only for web apps?

The core rules focus on web application security patterns because that's where the research data is strongest. Many rules (secrets management, input validation, command injection) apply broadly. Stack supplements extend coverage to specific ecosystems.

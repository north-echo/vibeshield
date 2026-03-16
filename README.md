# VibeShield

**Secure context rules for AI coding assistants**

## The Problem

AI coding tools generate code with predictable security vulnerabilities. Research shows that 45% of AI-generated code contains security flaws (Veracode), with a vulnerability rate 2.74x higher than human-written code (CodeRabbit). These failures cluster around specific, well-documented patterns like hardcoded credentials, broken authorization, and SSRF.

## What VibeShield Does

VibeShield is an open-source collection of security-focused context rules files for AI coding assistants. It ships as drop-in configuration files (`.cursorrules`, `CLAUDE.md`, etc.) that inject security constraints into the AI's generation context before code is written.

Prevention over detection — shift security left of the code, into the prompt context itself. Zero friction: copy a file, no installs required.

## Quick Start

### Claude Code
```bash
cp rules/CLAUDE.md your-project/CLAUDE.md
```

### Cursor
```bash
cp rules/.cursorrules your-project/.cursorrules
```

**Note:** Additional tool adapters (Copilot, Windsurf, Aider, Roo) coming in v0.2.

## What's Covered

VibeShield addresses 17 vulnerability patterns organized by frequency and impact:

### Tier 1 (Near-Universal)
- Broken authorization
- Hardcoded secrets
- Server-Side Request Forgery (SSRF)
- Client-side authentication/authorization

### Tier 2 (High Frequency)
- Insecure credential storage
- Missing input validation
- Business logic bypass
- Missing security headers
- Secrets in version control
- Permissive CORS

### Tier 3 (Context-Dependent)
- Dangerous deserialization
- Hallucinated dependencies
- Insufficient logging
- Information disclosure
- Weak cryptography
- Path traversal
- Command injection

## Project Structure

```
vibeshield/
├── core/
│   └── vibeshield-rules.md       # Canonical ruleset (tool-agnostic)
├── rules/
│   ├── CLAUDE.md                 # Claude Code adapter
│   └── .cursorrules              # Cursor adapter
├── stacks/                       # Stack-specific supplements (v0.2)
├── evidence/
│   ├── vulnerability-map.md      # V-ID to CVE/advisory cross-reference
│   └── sources.md                # Research citations
├── tests/
│   ├── test-prompts.md           # Prompts that should trigger rules
│   └── expected-behaviors.md     # What compliant output looks like
├── CONTRIBUTING.md
├── CHANGELOG.md
├── LICENSE                       # Apache 2.0
└── README.md
```

## What This Is NOT

- **Not a SAST scanner** — VibeShield works at generation time, not analysis time
- **Not a replacement for security testing** — Still run your scanners, penetration tests, and code reviews
- **Not a compliance framework** — This is developer tooling, not an audit checklist

## Stack Supplements

Stack-specific supplements are planned for v0.2:
- Node/Express
- Python (Django/Flask)
- Supabase
- Docker

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

Apache 2.0

## References

- [Vibe Security Radar](https://vibe-radar-ten.vercel.app/) — 130+ AI-linked CVEs tracked across 8 tools (March 2026)
- Veracode GenAI Code Security Report (2025) — 45% of AI-generated code contains flaws
- Tenzai Study (Dec 2025) — 69 vulnerabilities across 15 test apps built by 5 AI coding tools
- Escape.tech (2025) — 2,000+ vulnerabilities and 400+ exposed secrets in 5,600 vibe-coded apps
- CodeRabbit Analysis (Dec 2025) — 2.74x higher vulnerability rate in AI co-authored PRs
- Carnegie Mellon — AI-generated code is 61% functionally correct, 10.5% secure
- Unit 42 / Palo Alto Networks SHIELD Framework (Jan 2026)
- Kaspersky: Vibe Coding Security Risks (Oct 2025)
- Invicti: Security Issues in Vibe-Coded Web Apps
- OWASP Top 10 (2021), OWASP LLM Top 10 (2025)
